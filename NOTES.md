# Configuração da Infraestrutura

- [Lousa virtual](http://dontfile.com/curso513)
- [Repo do curso](https://github.com/4linux/513)

- [CLI do Vagrant](https://www.vagrantup.com/docs/cli/)

```console
vagrant up ansible-server
vagrant halt ansible-server
vagrant up
vagrant halt
vagrant destroy
vagrant destroy dbserver
vagrant up dbserver

vagrant ssh ansible-server # Preciso estar posicionado no dir em que está presente o Vagrantfile
```

# Conhecendo o Ansible

- Ferramentas de gerência de configuração (FGC)
  - Puppet e Chef são outras ferramentas
  - Provisionar: Ansible
  - Manter: Puppet
- Vem para substituir o método manual de configuração de ambientes
- Infraestrutura como código
  - Através das FGCs

- O Ansible é um framework open source que oferece recursos e ferramentas para implantar um modelo único de gerência de configurações em seu ambiente
- Quais são os ganhos com o Ansible?
  - Documentação instantânea;
  - Restore de backups e mudanças;
  - Processos bem definidos;
  - Ambiente padronizado;
  - Sistemas automatizados.

## A arquitetura do Ansible

<pre>
 Inventory                                    Componentes
    |                                         |__________ Server001
    V                                         |__________ Server002
 Playbook -> Ansible Config -> Python - SSH ->|__________ Server003
    ^                                         |__________ Server004
    |                                         |__________ Server005
 Modules                                      |__________ Server006
</pre>

- Vantagem: comunicação do Ansible com as outras máquinas  é realizada pelo SSH
  - Não é necessário um agente instalado nos servidores alvos

## Instalando o ansible no Linux

```console
# Acessar a máquina ansible-server
$ ssh suporte@172.16.0.200
Passwd: 4linux

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible -y
$ ansible --version

# Acessar o diretório do ansible
$ cd /etc/ansible
$ ls
ansible.cfg  hosts  roles
```

## Instalando o ansible no Windows
## Configurando o ansible (ansible.cfg)


```console
# Após uma determinada versão o ansible, é necessário gerar um arquivo modelo
$ sudo su
$ ansible-config init --disabled > ansible.cfg

# Editar
# Configurar as seguintes diretivas:
#  log_path=/var/log/ansible.log
#  private_key_file=/etc/keys/sshkeys
#  remote_user=root
#  roles_path=/etc/ansible/roles
#  timeout=30
$ vim ansible.cfg

```

## O SSH e chaves SSH no Ansible

```console
$ mkdir /etc/keys
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /etc/keys/sshkey
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /etc/keys/sshkey
Your public key has been saved in /etc/keys/sshkey.pub
The key fingerprint is:
SHA256:vEog7mA7iPWNKqeSJlzo4dMwavl0ICTDLNqS250dmto root@ansible-server
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|o                |
|++               |
|=+     .         |
|+.+.. . S        |
| Xo+.* . .       |
|X+Xo=+o .        |
|X%+++...         |
|*=BoE .          |
+----[SHA256]-----+

# Validar se a chave foi gerada
$ ls /etc/keys/
sshkey  sshkey.pub

# Enviar a chave pública para as máquinas alvo
$ ssh-copy-id -i /etc/keys/sshkey.pub ansible-server
$ ssh-copy-id -i /etc/keys/sshkey.pub webserver1
$ ssh-copy-id -i /etc/keys/sshkey.pub webserver2
$ ssh-copy-id -i /etc/keys/sshkey.pub dbserver

# Executar um comando via ssh sem precisar passar senha, passando a chave ssh
$ ssh -i /etc/keys/sshkey ansible-server cal
$ ssh -i /etc/keys/sshkey webserver1 cal
$ ssh -i /etc/keys/sshkey webserver2 cal
$ ssh -i /etc/keys/sshkey dbserver cal

```

## Configurar Inventários

```console
# Editar o invenvário
# Adicionar o conteúdo:
#   [local]
#   ansible-server
#   [webservers]
#   webserver[1:2]
#
#   [db]
#   dbserver
$ vim hosts

# Testar configuração
$ ansible local -m ping
ansible-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

$ ansible webservers -m ping
webserver2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
webserver1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

$ ansible db -m ping
dbserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

- Para nos comunicarmos com máquinas Windows:

```console
$ apt install pip -y
$ pip3 install pywinrm
Requirement already satisfied: pywinrm in /usr/lib/python3/dist-packages (0.3.0)

# Adicionar o seguinte conteúdo ao inventário:
#    [windows]
#    winclient
#    [windows:vars]
#    ansible_user=vagrant
#    ansible_password=vagrant
#    ansible_connection=winrm
#    ansible_port=5986
#    ansible_winrm_server_cert_validation=ignore
#    ansible_winrm_transport=ssl
$ vim hosts

# Acessar a máquina windows e no powershell executar:
$ ansible windows -m win_ping
winclient | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Ansible ad hoc

# Gerenciando Playbooks no Ansible
## Playbooks
## Variaveis
## Módulos (Ansible Module Index)
## Handlers

# Gerenciando Roles no Ansible
## Gerenciar Roles
## Tipos de Roles
## Variáveis
## Criar Role com variáveis

# Gerenciando Templates no Ansible
## Introdução a Templates
## Utilizar Roles com templates

# Gerenciando Testes com Kitchen
## Conhecendo o Kitchen
## Instalar o Kitchen
## Gerenciar Roles com Kitchen
## Gerenciar testes com Kitchen

# Gerenciando Testes com Molecule
## Conhecendo o Molecule
## Instalar o Molecule
## Gerenciar Roles com Molecule
## Gerenciar testes com Molecule

# Gerenciar ambientes com Ansible Galaxy
## Conhecendo o Ansible Galaxy
## Comandos essenciais do Ansible Galaxy
## Enviar Roles para o Github
## Sincronizar Roles do Github com Ansible Galaxy

# Gerenciar ambientes com AWX
## Conhecer o AWX
## Instalando o AWX
## Configurar Inventory/Credential/Projetos
## Configurar Job Templates
