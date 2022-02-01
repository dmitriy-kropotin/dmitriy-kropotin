0. спросил у коллеги про пакет менеджер в ol8.5. Оказалось вместо yum, используется dnf.
1. подключил репозиторий dev oracle: 
	>sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/x86_64/'
3. Установка VirtualBox: 
4. 	>sudo dnf update, sudo dnf install VirtualBox-6.1
6. подключил репозиторий vagrant: 
8. 
9.	 >sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
10. 
11. установил packer: 
12. 
13.	 >sudo dnf install packer
14. 
15. установка приложения git: 
16. клонирование репозитория на рабочую машину: 
17. сгенерировал ssh-key для подключения к github через ssh: 
18. добавил публичную часть ключа в настройки профиля github: 
19. 
20. 	>cat /home/tesla/.ssh/id_ed25519.pub
21. 
22. попытался запустить виртуальную машину командой vagrant up, получил ошибку No usable default provider could be found for your system.... погуглил, не сконфигурировал virtualbox. необходимо подключить модуль ядра. 
23. подключаю репозиторий sudo dnf config-manager --add-repo 'https://yum.oracle.com/repo/OracleLinux/OL8/developer/EPEL/x86_64/'
24. для конфигурирования virtualbox устанавливаю доп. компоненты: sudo dnf install kernel-devel gcc make perl bzip2 dkms. 
25. конфигурирования virtualbox: sudo /sbin/vboxconfig
26. попытка запуска вирутальнйо машины: vagrant up. Ура, успех
27. подключаюсь по к вирутальной машине: vagrant ssh
28. проверяю версию ядра: uname -r. версия - 3.10.0-1127.el7.x86_64
29. подключаю репозиторий ELREPO: sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org, sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
30. обновляю ядро на виртуальной машине: sudo yum --enablerepo elrepo-kernel install kernel-ml -y
31. обновляю конфу grub: sudo grub2-mkconfig -o /boot/grub2/grub.cfg, sudo grub2-set-default 0. перезагрузка виртуальной машины sudo reboot
32. проверяю версию ядра uname -r: 5.16.4-1.el7.elrepo.x86_64
33. узнаю версию centos at /etc/centos-release: CentOS Linux release 7.9.2009 (Core)
34. изучаю centos.json, редактирую версию CentOS: "artifact_version" : "7.9.2009", 	"image_name": "centos-7.9"
35. запускаю packer: packer build centos.json. не удачно, в конфигурации есть Deprecated configuration key: 'iso_checksum_type'. Убираю
36. запускаю packer: packer build centos.json. опять не удачно. 
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
