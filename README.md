# Hardening - Ansible

## Executando o playbook
Exemplo de Inventory
```
[aws]
127.0.0.1
#localhost ansible_connection=local

[aws:vars]
ansible_ssh_private_key_file = key/aws-cps.key
ansible_user = centos
```

Exemplo de uso:
```
ansible-playbook -i inventory playbook.yml
```

## Descrição

Esse playbook fornece inúmeras configurações relacionadas à segurança, fornecendo uma proteção completa com os padrões de segurança atuais. Ele é compatível com os sistemas operacionais RedHat, CentOS e Amazon Linux. A recomendação é que este playbook seja executado periodicamente em todos servidores da rede.
Resumo de configurações: 
* Configura a segurança do gerenciador de pacotes, como por exemplo a permissão de instalar somente pacotes assinados.
* Pacotes removidos por problemas de segurança conhecidos.
  * xinetd ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.1)
  * inetd ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.1)
  * tftp-server ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.5)
  * ypserv ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.4)
  * telnet-server ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.2)
  * rsh-server ([NSA](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm), Chapter 3.2.3)
  * prelink ([open-scap](https://static.open-scap.org/ssg-guides/ssg-centos7-guide-standard.html))
* Módulos pam e pam_limits configurados para impedir vulnerabilidades conhecidas.
* Configura as permissões dos paths do sistema.
* Desativar o core dump via soft limits.
* Restringir logins do root ao System Console. 
* Arquivos imutáveis que não podem ser modificados, gravados, removidos e renomeados. OBS.: Para desabilitar a flag imutável deve executar o comando chattr -i no arquivo.
  * /etc/passwd
  * /etc/group
  * /etc/shadown
  * /etc/gshadown
  * /etc/services
  * /etc/sudoers
* Configura os parâmetros do kernel via sysctl.
* Instala e configura o auditd.
* Configura diversas medidas de segurança no ssh-client e ssh-server.
* OBS.: O login do root via SSH é desabilitado! Portanto sempre será preciso de um usuário com permissões de su ou sudo para se autenticar via SSH.
* O serviço de SFTP foi desabilitado, prefira usar o SCP para transferências de arquivos.
* O playbook não atualiza pacotes do sistema e não instala patches de segurança.
 
## Variáveis

| Nome           | Valor Padrão | Descrição                        |
| -------------- | ------------- | -----------------------------------|
| `os_env_extra_user_paths`| [] | Adiciona PATHs adicionais ao sistema. (default é vazio).|
| `os_env_umask`| 027| Configura a permissão padrão de novos arquivos para `750` |
| `os_auth_pw_max_age`| 60 | Tempo máximo de senhas (configure para `99999` para efetivamente desabilitar isso) |
| `os_auth_pw_min_age`| 7 | Tempo mínimo de senhas (antes de permitir qualquer outra alteração de senhas)|
| `os_auth_retries`| 5 | O número máximo de tentativas de autenticação antes do login ser bloqueado.|
| `os_auth_lockout_time`| 600 | Tempo em segundos que precisa passar, se a conta foi bloqueada devido a muitas tentativas de autenticação com falha|
| `os_auth_timeout`| 60 | tempo limite de autenticação em segundos; portanto, o login será encerrado se esse tempo passar|
| `os_auth_allow_homeless`| false | true se permitir que usuários sem home façam login|
| `os_auth_pam_passwdqc_enable`| true | true se você deseja usar a verificação de senha forte no PAM usando passwdqc|
| `os_security_users_allow`| [] | lista de coisas que um usuário pode fazer. Pode conter `change_user`.|
| `os_security_kernel_enable_module_loading`| true | true se você quiser alterar os módulos do kernel quando o sistema estiver em execução (ex `modprobe`, `rmmod`)|
| `os_security_kernel_enable_core_dump`| false | true faz com que o kernel ao estar falhando ou se comportando mal gere um dump do núcleo do kernel. |
| `os_security_suid_sgid_enforce`| true | true se você deseja retirar os bits SUID/SGID. Já existe uma lista de itens que são configurados, mas você também pode adicionar seus próprios itens.|
| `os_security_suid_sgid_blacklist`| [] | uma lista de paths que devem ter seus bits SUID/SGID retirados.|
| `os_security_suid_sgid_whitelist`| [] | uma lista de caminhos que não devem ter seus bits SUID/SGID retirados.|
| `os_security_suid_sgid_remove_from_unknown`| false | true se você deseja retirar bits SUID/SGID de qualquer arquivo que não esteja explicitamente configurado em uma lista negra. Isso fará com que todas as pesquisas executadas pelo Ansible pelos sistemas de arquivos montados procurem bits SUID/SGID que não estão configurados na lista negra padrão e do usuário. Se encontrar um bit SUID/SGID, ele será retirado, a menos que esse arquivo esteja na sua lista de permissões.|

