# DUAL BOOT CON WINDOWS XX #

Editado por última vez: **8/1/2022 11:40 AM**

Instalación de ArchLinux:
    
1. Cargar la distribución de teclado correspondiente. Por defecto, la distribución es US (Inglés). Para listar las distribuciones de teclado disponibles usar:

        ls /usr/share/kbd/keymaps/**/*.map.gz
    
    *Si se desea cargar la distribución para un teclado en español por ejemplo, usar:*
   
        loadkeys es        

2. En caso de tener sólo wifi, usar la utilidad:

        ip link (Para listar las interfaces. Ubicar la de Wifi, generalmente es wlp2s0)
        iwctl
        station <wlan> connect "nombre de la red" [pass]

    *Seleccionar la red, e ingresar contraseña.*

3. Activar la sincronización del reloj del sistema con Internet: 

        timedatectl set-ntp true

4. Verificar: (opcional)

        timedatectl status

5. Identificar los discos:

        lsblk

6. Verificar la tabla de particiones: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

7. Crear particion swap :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +2G
        8200
        W
        Y
        
8. Crear particion / :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        ENTER
        8304
        W
        Y

9. Verificar:

        lsblk

10. Formatear particion swap :

        mkswap /dev/sda5

11. Activar swap :

        swapon /dev/sda5

12. Formatear particion / :

        mkfs.ext4 /dev/sda6

13. Montar particion / en /mnt :
        
        mount /dev/sda6 /mnt

14. Crear directorio para /boot :

        mkdir -p /mnt/boot

15. Montar partición /boot, en este caso es /dev/sda2 pero puede cambiar, se debe usar la partición EFI de Windows:

        mount /dev/sda2 /mnt/boot

16. Instalar los paquetes base:

        pacstrap /mnt base linux linux-firmware nano

17. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

18. Iniciar sesión como root en la instalación:

        arch-chroot /mnt

19. Configurar zona horaria:

        ln -sf /usr/share/zoneinfo/America/Havana /etc/localtime

20. Generar /etc/adjtime:

        hwclock --systohc

21. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8
        
22. Construir el soporte de idioma: 

        locale-gen

23. Crear el archivo de configuración correspondiente:

        echo LANG=en_US.UTF-8 > /etc/locale.conf
   
24. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo:*

        echo arch > /etc/hostname

25. Agregar el hostname a /etc/hosts, por ejemplo:

        nano /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando arch por tu hostname*
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        arch.localdomain               arch

26. Establecer contraseña para  root:

        passwd

27. Ahora puedes crear tu usuario:

        useradd -m username
        passwd username
        usermod -aG wheel,video,audio,storage username

28. Para privilegios de superusuario necesitamos sudo:

        pacman -S sudo

29. Edita el "Sudoers file" con nano y descomenta la línea con "wheel":

        EDITOR=nano visudo
        "# %wheel ALL=(ALL) ALL"

30. Instalar **systemd-boot**: (En este paso se puede instalar GRUB o otro cargador a gusto)

        bootctl --path=/boot install

31. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 3
        editor 0

32. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda6) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

33. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

34. Antes de reiniciar instalar el administrador de red:

        pacman -S networkmanager
        systemctl enable NetworkManager

35. Salir de la sesión, desmontar particiones y reiniciar:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado
        reboot

Copiado y modificado de https://wiki.archlinux.org/title/installation_guide