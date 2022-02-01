0. спросил у коллеги про пакет менеджер в ol8.5. Оказалось вместо yum, используется dnf.
1. подключил репозиторий dev oracle: 
	>sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/x86_64/'
2. Установка VirtualBox: 
 	>sudo dnf update
 	>
 	>sudo dnf install VirtualBox-6.1
 	
. подключил репозиторий vagrant: 
	 >sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
. установил packer: 
	 >sudo dnf install packer
. установка приложения git: 
. клонирование репозитория на рабочую машину: 
. сгенерировал ssh-key для подключения к github через ssh: 
. добавил публичную часть ключа в настройки профиля github: 
	>cat /home/tesla/.ssh/id_ed25519.pub
. попытался запустить виртуальную машину командой 
	>vagrant up
. получил ошибку 
 	>No usable default provider could be found for your system.... 
. погуглил, не сконфигурировал virtualbox. необходимо подключить модуль ядра. 
. подключаю репозиторий 
 	>sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/EPEL/x86_64/'
. для конфигурирования virtualbox устанавливаю доп. компоненты: 
	>sudo dnf install kernel-devel gcc make perl bzip2 dkms. 
. конфигурирования virtualbox: 
	>sudo /sbin/vboxconfig
. попытка запуска вирутальнйо машины: 
	>vagrant up. 
Ура, успех
. подключаюсь по к вирутальной машине: 
	>vagrant ssh
. проверяю версию ядра: 
	>uname -r. 
	>3.10.0-1127.el7.x86_64
. подключаю репозиторий ELREPO: 
	>sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	>sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
. обновляю ядро на виртуальной машине: 
	>sudo yum --enablerepo elrepo-kernel install kernel-ml -y
. обновляю конфу grub: 
	>sudo grub2-mkconfig -o /boot/grub2/grub.cfg
	>sudo grub2-set-default 0
. перезагрузка виртуальной машины 
	>sudo reboot
. проверяю версию ядра 
	>uname -r
	> 5.16.4-1.el7.elrepo.x86_64
. узнаю версию centos 
	>cat /etc/centos-release
	>CentOS Linux release 7.9.2009 (Core)
. изучаю centos.json, редактирую версию CentOS: 
	>"artifact_version" : "7.9.2009"
	>"image_name": "centos-7.9"
40. запускаю packer: packer build centos.json. не удачно, в конфигурации есть Deprecated configuration key: 'iso_checksum_type'. Убираю
41. запускаю packer: packer build centos.json. опять не удачно. 
	==> centos-7.9: Error starting VM: VBoxManage error: VBoxManage: error: The virtua
	l machine 'packer-centos-vm' has terminated unexpectedly during startup because of signal 6
	==> centos-7.9: VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component MachineWrap, interface IMachine
24. на слаке уже обсудили, что virtualbox по-умолчанию запускает gui, а я запускаю через ssh. добавляю параметр "headless":"true"
25. запускаю packer: packer build centos.json. успех! заняло около 15 минут.
26. импортирую в vagrant: vagrant box add --name centos-7.9 centos-7.9.2009-kernel-5-x86_64-Minimal.box, 
27. тестирую: mkdir test, cd test, vagrant init centos-7.9, vagrant up
28. проверяю версию ядра: vagrant ssh, 
	[vagrant@localhost ~]$ uname -r
	5.16.4-1.el7.elrepo.x86_64
29. загружаю box в vagrant cloud vagrant cloud publish --release dmitriy-kropotin/centos-7-9 1.0 virtualbox \centos-7.9.2009-kernel-5-x86_64-Minimal.box. Ссылка - https://app.vagrantup.com/dmitriy-kropotin/boxes/centos-7-9
