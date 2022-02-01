0. спросил у коллеги про пакет менеджер в ol8.5. Оказалось вместо yum, используется dnf.
1. подключил репозиторий dev oracle: 
	>sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/x86_64/'
2. Установка VirtualBox: 
 	>sudo dnf update
 	>
 	>sudo dnf install VirtualBox-6.1
3. подключил репозиторий vagrant: 
	 >sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
4. установил packer: 
	 >sudo dnf install packer
5. установка приложения git: 
 	>sudo dnf install git-all
6. сгенерировал ssh-key для подключения к github через ssh:
	>ssh-keygen -t ed25519 -C "dmitriy.kropotin@realize.dev"
7. вывод публичного ключа для профиля в github: 
	>cat /home/tesla/.ssh/id_ed25519.pub
8. клонирования репозитория из github:
	>git clone git@github.com:dmitriy-kropotin/manual_kernel_update.gi
9. попытался запустить виртуальную машину командой 
	>vagrant up
10. получил ошибку 
 	>No usable default provider could be found for your system.... 
11. погуглил, не сконфигурировал virtualbox. необходимо подключить модуль ядра. 
12. для этого подключаю репозиторий epel: 
 	>sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/EPEL/x86_64/'
13. устанавливаю модули ядра и доп. по : 
	>sudo dnf install kernel-devel gcc make perl bzip2 dkms. 
14. конфигурирования virtualbox: 
	>sudo /sbin/vboxconfig
15. попытка запуска вирутальнйо машины: 
	>vagrant up. 
Ура, успех
16. подключаюсь по к вирутальной машине: 
	>vagrant ssh
17. проверяю версию ядра: 
	>uname -r. 
	>3.10.0-1127.el7.x86_64
18. подключаю репозиторий ELREPO: 
	>sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	>sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
19. обновляю ядро на виртуальной машине: 
	>sudo yum --enablerepo elrepo-kernel install kernel-ml -y
20. обновляю конфу grub: 
	>sudo grub2-mkconfig -o /boot/grub2/grub.cfg
	>sudo grub2-set-default 0
21. перезагрузка виртуальной машины 
	>sudo reboot
22. проверяю версию ядра 
	>uname -r
	> 5.16.4-1.el7.elrepo.x86_64
23. узнаю версию centos 
	>cat /etc/centos-release
	>CentOS Linux release 7.9.2009 (Core)
24. изучаю centos.json, редактирую версию CentOS: 
	>"artifact_version" : "7.9.2009"
	>"image_name": "centos-7.9"
25. запускаю packer: 
	>packer build centos.json
не удачно, в конфигурации есть 
	>Deprecated configuration key: 'iso_checksum_type'
убираю этот параметр.
26. запускаю packer: packer build centos.json. опять не удачно. 
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
