Editado por última vez: **9/1/2022 2:30 PM**

# Instalación de ArchLinux Dual Boot: #
  - [Solo](https://github.com/alain-david/arch-dual-boot/blob/main/README.md)
  - [Dual](https://github.com/alain-david/arch-dual-boot/blob/main/README_DUAL.MD)
    
0. En caso de tener sólo wifi, usar la utilidad:

        ip link (Para listar las interfaces. Ubicar la de Wifi, generalmente es wlp2s0)
        iwctl
        station <wlan> connect "nombre de la red" [pass]

    *Seleccionar la red, e ingresar contraseña.*

1. Cargar la distribución de teclado correspondiente. Por defecto, la distribución es US (Inglés). Para listar las distribuciones de teclado disponibles usar:

        ls /usr/share/kbd/keymaps/**/*.map.gz
    
    *Si se desea cargar la distribución para un teclado en español por ejemplo, usar:*
   
        loadkeys es        

2. Activar la sincronización del reloj del sistema con Internet: 

        timedatectl set-ntp true

3. Verificar: (opcional)

        timedatectl status

4. Identificar los discos:

        lsblk

5. Verificar la tabla de particiones: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

6. Crear particion swap :

        gdisk /dev/sda
        n       #Nueva 
        5       #Particion 
        ENTER   #Sector
        +2G     #Espacio
        8200    #Tipo 8200="swap"
        W
        Y
        
7. Crear particion / :

        gdisk /dev/sda
        n
        6       #Particion
        ENTER   #Sector
        ENTER   #Espacio
        8304    #Tipo 8304="/"
        W
        Y

8. Verificar:

        fdisk -l

9. Formatear particion swap :

        mkswap /dev/sda5

10. Activar swap :

        swapon /dev/sda5

11. Formatear particion / :

        mkfs.ext4 /dev/sda6

12. Montar particion / en /mnt :
        
        mount /dev/sda6 /mnt

13. Crear directorio para /boot :

        mkdir /mnt/boot

14. Montar partición /boot, en este caso es /dev/sda2 pero puede cambiar, se debe usar la partición EFI de Windows:

        mount /dev/sda2 /mnt/boot

15. Instalar los paquetes base:

        pacstrap /mnt base linux linux-firmware nano

16. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

17. Iniciar sesión como root en la instalación:

        arch-chroot /mnt

18. Configurar zona horaria:

        ln -sf /usr/share/zoneinfo/America/Havana /etc/localtime

19. Generar /etc/adjtime:

        hwclock --systohc

20. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8
        
21. Construir el soporte de idioma: 

        locale-gen

22. Crear el archivo de configuración correspondiente:

        echo LANG=en_US.UTF-8 > /etc/locale.conf
   
23. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo:*

        echo arch > /etc/hostname

24. Agregar el hostname a /etc/hosts, por ejemplo:

        nano /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando arch por tu hostname*
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        arch.localdomain               arch

25. Establecer contraseña para  root:

        passwd

26. Ahora puedes crear tu usuario:

        useradd -m username
        passwd username
        usermod -aG wheel,video,audio,storage username

27. Para privilegios de superusuario necesitamos sudo:

        pacman -S sudo

28. Edita el "Sudoers file" con *nano* y descomenta la línea con "wheel":

```bash
    EDITOR=nano visudo
    ## Uncomment to allow members of group wheel to execute any command
    # %wheel ALL=(ALL) ALL
```
29. Instalar **systemd-boot**: (En este paso se puede instalar GRUB o otro cargador a gusto)

        bootctl --path=/boot install

30. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 3
        editor 0

31. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda6) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

32. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

33. Antes de reiniciar instalar el administrador de red:

        pacman -S networkmanager
        systemctl enable NetworkManager

34. Salir de la sesión, desmontar particiones y reiniciar:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado
        reboot

Copiado y modificado de https://wiki.archlinux.org/title/installation_guide