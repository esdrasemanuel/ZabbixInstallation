# ZabbixInstallation
installation of Zabbix and configuration of alerts via email.


instalação do Zabbix e configuraçao de alertas via email.

1ª Etapa: configurar o Postfix:
instalar:
apt-get install postfix s-nail
 
Ao ser perguntado, escolha a instalação "Internet Site"

Crie um link simbólico para a ferramenta "mail" instalada pelo s-nail
ln -s /usr/bin/mail /bin/mail

Em seguida, entre no diretório de configuração do Postfix, faça o backup do arquivo de configuração:

# cd /etc/postfix/
# mv main.cf main.cf.old

Nele, coloque somente as seguintes linhas:

#SMTP relayhost
relayhost = [smtp.gmail.com]:587
# TLS Settings
smtp_tls_loglevel = 1
smtp_use_tls = yes
smtpd_tls_received_header = yes
# TLS
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous 


Crie o arquivo "sasl_passwd", contendo o servidor SMTP do Google e a conta que será utilizada para envio dos e-mails.

Dessa forma:

# vi sasl_passwd

[smtp.gmail.com]:587 zabbixtestemail@gmail.com:Senha 

Em seguida, rodamos o comando "postmap" no arquivo "sasl_passwd" e no "main.cf", para que eles possam ser reconhecidos e utilizados pelo Postfix:

# sudo postmap /etc/postfix/sasl_passwd; sudo postmap /etc/postfix/main.cf

Reinicie o serviço do Postfix:

# service postfix restart

Para testar (troque "fulano" pelo seu login no gmail):

echo "Este e' um e-mail de teste!" | mail -s "Teste" fulano@gmail.com

verifique a fila de saída de e-mails:
mailq

2ª Etapa: Configurando o Zabbix
Agora veja como configurar o Zabbix, através de sua interface/front-end, para que ele envie alertas de e-mails, usando o servidor SMTP do Google, através do Postfix que acabamos de configurar.

Acesse a interface web do Zabbix:

    http://ip_do_servidor/zabbix 


Faça o "login" e clique em: Administration → Media Types → Email

Em seguida, configure da seguinte forma:

    Type :: Email
    SMTP server :: IP da máquina que foi configurado o Postfix
    SMTP helo :: smtp.gmail.com
    SMTP email :: zabbixtestemail@gmail.com (lembre de trocar)
    Enabled :: Deixe Habilitado (default) 


Clique em: "Save"

Para testar se o Zabbix enviará os alertas via e-mail, é só criar uma Action e induzir o alerta, ativando uma trigger qualquer, por exemplo.

Nova ação

Entregar notificaões é uma das coisas que o recurso de ações do Zabbix faz. Portanto, para configurar uma notificação, acesse Configuração → Ações e clique em Criar ação.

Neste formulário, informe o nome da ação, conforme imagem a seguir. 

Neste mesmo formulário são apresentadas algumas macros, tais quais {TRIGGER.STATUS} e {TRIGGER.NAME}. Existem macros sendo utilizadas tanto no campo Assunto padrão quanto no campo Mensagem padrão. O valor destas macros será substituído pelo valor atual em tempo de execução. 

 Simplificando, se nós não adicionarmos nenhuma condição mais específica, a ação irá ser executada sempre que ocorrer uma mudança de estado em uma trigger (seja indo para o estado de 'Ok' ou de 'Incidente').

Além de definir o nome, o conteúdo da mensagem e as condições em que a ação deverá ser acionada nós precisamos também definir o que deverá ser feito. Isso é configurado na aba Ações. Clique no link Nova dentro da caixa de operações da ação. 

 Agora, clique no link Adicionar dentro da caixa Enviar para usuários marque e selecione o usuário que criamos anteriormente. Selecione a opção 'Email' como valor para o campo Enviar apenas para. Quando terminar, clique no link Adicionar no bloco de detalhes da operação.

Isso é tudo que precisa ser configurado em uma ação simples de notificação. Clique no botão Adicionar do formulário de ações. 

Mídia de usuário

Para definir o endereço específico de cada usuário:

    Acesse Administração → Usuários
    Abra o formulário de propriedades do usuário
    Na aba Mídia, clique no botão Adicionar


