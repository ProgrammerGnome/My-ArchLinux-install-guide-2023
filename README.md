# My ArchLinux installation guide [HU]

## Vanilla Arch Telepítési Útmutató (2023.04.)

### Magyar billentyűzet
```
	loadkeys hu
	## loadkeys uk ##
```
### Wifi Beallitasa (opcionális)
```
	iwctl
	#interaktiv prompt
		device list
		station wlan0 scan
		station wlan0 get-networks
		station wlan0 connect TE_HALOZATOD
		exit
	ping google.com
```
### Szinkronizálás távoli idő kiszolgálóval
```
	timedatectl set-ntp true
```
### Lemezek megtekintése
```
lsblk
```
### Partícionálás
```
	cfdisk 
	több lemez esetén: cfdisk/dev/sdXXX 			(XXX == ROOT lemez)
```
### ROOT partíció formázása ext4-re
```
	mkfs.ext4 /dev/sdaXX (ROOT)				(Pl.: sda1 és sda2)
```  
### GYÖKÉR Csatolása
```
	mount /dev/sdaXX (ROOT) /mnt				(Pl.: sda1)
``` 
### Külön HOME könyvtár létrehozása, formázása, csatolása
```
	mkdir /mnt/home
	mkfs.ext4 /dev/xxx (HOME)				(Pl.: xxx = sda2)
	mount /dev/xxx /mnt/home
```
### MIRROR beállítása

##### MANUÁLIS:
```
	mcedit /etc/pacman.d/mirrorlist
```
##### AUTOMATIKUS:
```
	pacman -Syy
	pacman -S reflector
	cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak (Visszaállítás, ha kell!)
	reflector -c "US" -f 12 -l 10 -n 12 --save/etc/pacman.d/mirrorlist
```
### TELEPÍTÉS (alap rendszer + pár hasznos dolog)
```
	pacstrap /mnt base base-devel linux linux-headers linux-firmware mc bash-completion gvfs ntfs-3g btrfs btrfs-progs git

	pacstrap /mnt base linux linux-firmware vim nano	
```
(Megj.: A továbbiakban mcedit helyett használd azt, amit itt feltelepítettél.)

(Megj.: A kettő parancs alkalmazásaiból egyénileg is válogathatsz.)
      
### FSTAB generálás
```
	genfstab -U /mnt >> /mnt/etc/fstab
```
### TELEPÍTETT rendszer átvétele
```
	arch-chroot /mnt
```
### HELYI időzóna beállítása
```
	ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime
```
### RENDSZER idő -> a hardver órába
```
	hwclock --systohc
```
### LOCALE generálás 
```
	mcedit /etc/locale.gen
	hu_HU.UTF-8
	en_US.UTF-8

	locale-gen
```
### MAGYAR lokális formátumok beállítása
```
	mcedit /etc/locale.conf
	LANG=hu_HU.UTF-8
	LC_TIME=hu_HU.UTF-8
```
### EGY MÁSIK MEGOLDÁS (LOCALE generálás + MAGYAR lokális formátumok beállítása)
```
	locale-gen

	mcedit /etc/locale.gen
	hu_HU.UTF-8
	en_US.UTF-8

	echo LANG=hu_HU.UTF-8 >/etc/locale.conf
	export LANG=hu_HU.UTF-8
```
### MAGYAR Paranccsori billentyűkiosztás
```
	mcedit /etc/vconsole.conf
	KEYMAP=hu
	## KEYMAP=uk ##
	FONT=lat2-16
	FONT_MAP=8859-2
```
### HOSTNÉV beállítása
```
	echo EGYÉNI_GÉPNÉV > /etc/hostname
```
### HOST fájl beállítása
```
	mcedit /etc/hosts	
	127.0.0.1	localhost
	::1			localhost
	127.0.0.1	EGYÉNI_GÉPNÉV.localdomain	EGYÉNI_GÉPNÉV
```
#####	------- EZ MEG EGY MÁSIK MEGOLDÁS: ------
```
	touch /etc/hosts
	127.0.0.1      localhost
	::1             	localhost
	127.0.1.1       EGYÉNI_GÉPNÉV
```
### NETWORK manager telepítése
```
	pacman -S networkmanager network-manager-applet
```
### NETWORK manager engedélyezése
```
	systemctl enable NetworkManager
```
### ROOT jelszó létrehozása
```	
	passwd
```
### RENDSZERBETÖLTŐ telepítése

#####	(„EGYSZERŰ MÓD”)
```
	pacman -S grub
	grub-install /dev/sda
	
	mcedit /etc/default/grub
	GRUB_DISABLE_OS_PROBER=false
	
	grub-mkconfig -o /boot/grub/grub.cfg
```
##### (EFI MÓD)
```
	pacman -S grub efibootmgr os-prober
```
##### OS-PROBER ENGEDÉLYEZÉSE (Értelemszerűen csak EFI mód esetén.)
```
	mcedit /etc/default/grub
	GRUB_DISABLE_OS_PROBER=false 
	
	grub-install --target=x86_64-efi–efi-directory=/boot/efi
	--- VAGY: ---
	grub-install --target=x86_64-efi --bootloader-id=GRUB –efi-directory=/boot/efi

	--- ÉS: ---
	grub-mkconfig -o /boot/grub/grub.cfg
```
### KILÉPÉS és újraindítás
```	
	exit
	umount -R /mnt
	reboot			#Közben a boot sorrendet ne felejtsd el visszaállítani!
```
### WIFI beállítása (opcionális)
```
	nmcli device wifi list
	nmcli device wifi connect A_TE_HALOZATOD password A_TE_JELSZAVAD
```
### MULTILIB bekapcsolása
```
	mcedit /etc/pacman.conf

	[multilib]		
  
	pacman -Syu
	include = /etc/pacman.d/mirrorlist
```
=> Majd FRISSÍTÉS (Ellenőrizni a multilib letöltődését!)

### Chaotic-AUR letöltése és bekapcsolása
```
sudo pacman-key --recv-key FBA220DFC880C036 --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key FBA220DFC880C036
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst' 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'

sudo nano /etc/pacman.conf 			(Ebbe a fájlba értelemszerűen beleíruk a következő két sor tartalmát.)

[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

### SAJÁT felhasználó hozzáadása
```
	useradd -m -g users -G audio,video,network,wheel,storage,rfkill -s /bin/bash VAR

	usermod -aG wheel,audio,video,storage VAR	#(Megj.: Ez a 2023-as verzió.)
```
Ahol: FELHASZNÁLÓNÉV = VAR


### JELSZÓ létrehozása a felhasználóhoz
```
	passwd FELHASZNÁLÓNÉV
```
### SUDO bekapcsolása a felhasználóhoz.
```
	EDITOR=mcedit visudo
  
	#%wheel ALL=(ALL) ALL (hastaget kivenni)
```
### KILÉPÉS, és felhasználóval történő bejelentkezés
```
	exit
```
### XORG letöltése
```
	sudo pacman -S xorg-server xorg-appres xorg-xinit lxterminal
```
### BEJELENTKEZÉS KEZELŐ letöltése és engedélyezése
```
	sudo pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
	sudo systemctl enable lightdm.service -f
	sudo systemctl set-default graphical.target
```
*******************************************************
### NE INDÍTSD ÚJRA AMÍG NINCS TELEPÍTETT ASZTAL!
*******************************************************

### MAGYAR billentyű az Xorg-hoz + CTRL+ALT+BACKSPACE X resethez
```
mcedit /etc/X11/xorg.conf.d/00-keyboard.conf
```

```
Section "InputClass"
	Identifier "keyboard"
	MatchIsKeyboard "yes"
	Option "XkbLayout" "hu"
	Option "XkbVariant" "nodeadkeys"
	Option "XkbOptions" "terminate:ctrl_alt_bksp"
EndSection
```
*******************************************************

### ASZTALI KÖRNYEZET TELEPÍTÉSE
```
	sudo pacman -S lxde		(Ez egy javaslat, igazából ehelyett bármilyen asztali környezetet telepíthetsz!)
```
#### Miért az LXDE?

##### Gyors, kis RAM -és energiaigényű és jól konfigurálható (letisztult, esztétikus).
