Editado por última vez: **29/1/2022 20:08 PM**

0. En caso de tener sólo wifi, usar la utilidad:

        ip link (Para listar las interfaces. Ubicar la de Wifi, generalmente es wlp2s0)
        iwctl
        station "wlan" connect "nombre de la red"

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

5. Crear particion EFI (En dualboot omitir este paso):

        gdisk /dev/sda
        n       #Nueva
        ENTER   #Particion 
        ENTER   #Sector
        +200M   #Espacio
        ef00    #Tipo ef00="EFI"
        W
        Y

6. Crear particion swap :

        gdisk /dev/sda
        n       #Nueva
        ENTER   #Particion 
        ENTER   #Sector
        +2G     #Espacio
        8200    #Tipo 8200=swap
        W
        Y
        
7. Crear particion / :

        gdisk /dev/sda
        n
        ENTER   #Particion
        ENTER   #Sector
        ENTER   #Espacio
        8304    #Tipo 8304="/"
        W
        Y

8. Verificar:

        fdisk -l        

9. Formatear particion EFI (En dualboot omitir este paso):

        mkfs.fat -F32 /dev/sdax

10. Formatear particion swap :

        mkswap /dev/sdax

11. Activar swap :

        swapon /dev/sdax

12. Formatear particion / :

        mkfs.ext4 /dev/sdax

13. Montar particion / en /mnt :
        
        mount /dev/sdax /mnt

14. Crear directorio para /boot :

        mkdir /mnt/boot

15. Montar partición /boot :

        mount /dev/sda1 /mnt/boot

16. Instalar los paquetes base:

        pacstrap /mnt base linux linux-firmware nvim

17. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

18. Iniciar sesión como root en la instalación:

        arch-chroot /mnt

19. Configurar zona horaria:

        ln -sf /usr/share/zoneinfo/America/Havana /etc/localtime

20. Generar /etc/adjtime:

        hwclock --systohc

21. Generar locales:

        nvim /etc/locale.gen

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

        nvim /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando arch por tu hostname*
        
        127.0.0.1        localhost
        ::1              localhost
        127.0.1.1        arch.localdomain               arch

26. Establecer contraseña para  root:

        passwd

27. Ahora puedes crear tu usuario:

        useradd -m username
        passwd username
        usermod -aG wheel,video,audio,storage username

28. Para privilegios de superusuario necesitamos sudo:

        pacman -S sudo

29. Edita el "Sudoers file" con *nano* y descomenta la línea con "wheel":

```bash
    EDITOR=nvim visudo
    ## Uncomment to allow members of group wheel to execute any command
    # %wheel ALL=(ALL) ALL
```
30. Instala **GRUB**:

        pacman -S grub efibootmgr os-prober
        grub-install --target=x86_64-efi --efi-directory=/boot

31. Configurar os-prober en /etc/default/grub:    
```bash
    ## Descomentar la siguiente linea:
    #GRUB_DISABLE_OS_PROBER=false
```
32. Ahora puedes ejecutar:

        grub-mkconfig -o /boot/grub/grub.cfg

32. Antes de reiniciar instalar el administrador de red:

        pacman -S networkmanager
        systemctl enable NetworkManager

33. Salir de la sesión, desmontar particiones y reiniciar:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado
        reboot

Copiado y modificado de https://wiki.archlinux.org/title/installation_guide
