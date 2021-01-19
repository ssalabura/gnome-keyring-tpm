# Szyfrowanie systemu z *dm-crypt* w scenariuszu *LUKS on a partition*

Wygląd tablicy partycji w tym scenariuszu jest następujący:

| Boot partition        | LUKS2 encrypted system partition | Optional free space for additional partitions to be set up later  |
|-----------------------|----------------------------------|-------------------------------------------------------------------|
|/dev/sda1              | /dev/sda2                        |                                                                   |

W celu zaszyfrowania systemu najlepiej zainstalować Arch Linuxa od początku. W tym celu wykonujemy po kolei standardowe kroki instalacji aż do momentu podziału dysku na partycje.

## Bezpieczne czyszczenie dysku z danych
Dla bezpieczeństwa, na początku zaleca się bezpiecznie wyczyścić dane z szyfrowanego dysku przez nadpisanie ich losowymi danymi. 
Zgodnie z [wiki Arch Linuxa](https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation), należy w tym celu wykonać kolejno:
```
cryptsetup open --type plain -d /dev/urandom /dev/<block-device> to_be_wiped
```
gdzie \<block-device\> jest nazwą dysku w postaci sdX, który chcemy wyczyścić. 
Następnie za pomocą ```lsblk``` upewniamy się, że powyższa komenda utworzyła tymczasowy kontener _to_be_wiped_ na odpowiedniej partycji.
Dalej, wykonujemy czyszczenie dysku:
```
dd if=/dev/zero of=/dev/mapper/to_be_wiped status=progress
```
i usuwamy tymczasowy kontener utworzony na potrzeby czyszczenia:
```
cryptsetup close to_be_wiped
```

## Tworzenie partycji
W tym scenariuszu będziemy potrzebować co najmniej partycji ```/``` z systemem i plikami użytkownika, oraz partycji ```/boot```, 
która nie będzie szyfrowana i będzie służyła do rozruchu. Można utworzyć więcej zwykłych partycji, jednak każda z nich
będzie miała osobne hasło.

## Przygotowanie zwykłych partycji (innych niż /boot)
Jeśli nie chcemy zmieniać domyślnych parametrów szyfrowania, takich jak długość klucza), to wykonujemy kolejno, upewniając się, że sda2 to partycja, którą chcemy zaszyfrować:
```
cryptsetup -y -v luksFormat /dev/sda2
cryptsetup open /dev/sda2 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
```
Przy okazji, zostaniemy zapytani o hasło do dysku, które chcemy ustawić.
W celu zmiany domyślnych parametrów można zapoznać się z [opcjami dla szyfrowania](https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption#Encryption_options_for_LUKS_mode)

Dalej sprawdzamy, czy montowanie i mapowanie dysku działa tak, jak powinno:
```
umount /mnt
cryptsetup close cryptroot
cryptsetup open /dev/sda2 cryptroot
mount /dev/mapper/cryptroot /mnt
```

## Partycja boot
Tę partycję przygotowujemy normalnie, tzn. dla systemu bez EFI:
```
mkfs.ext4 /dev/sda1
```
lub dla systemu z EFI:
```
mkfs.fat -F32 /dev/sda1
```
a następnie
```
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## Dalsza instalacja
Teraz, postępujemy dalej tak samo, jak przy zwykłym instalowaniu Archa z dwoma wyjątkami. Będziemy jeszcze musieli skonfigurować mkinitcpio oraz bootloader.

## mkinitcpio
Gdy wykonamy już chroot do nowego systemu, należy edytować plik ```/etc/mkinitcpio.conf``` tak, aby linia zawierająca HOOKS wyglądała następująco:
```
#/etc/mkinitcpio.conf
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt filesystems fsck)
```
W szczególności, należy się upewnić, że na liście znajdują się **udev**, **keyboard**, **keymap** i **encrypt**

Po edycji tego pliku należy zgodnie z tutorialem na wiki Archlinuxa wykonać
```
mkinitcpio -P
```

## bootloader
W konfiguracji bootloadera należy zadbać o to, aby ustawić następujący parametr kernela:
```
cryptdevice=UUID=device-UUID:cryptroot root=/dev/mapper/cryptroot
```
gdzie device-UUID jest UUID partycji systemowej /dev/sda2

W przypadku, gdy bootloader to **GRUB**, wystarczy edytować plik ```/etc/default/grub``` i wykonać następujące czynności:
* odkomentować linię ```GRUB_ENABLE_CRYPTODISK=y```
* edytować linię ```GRUB_CMDLINE_LINUX_DEFAULT="..."``` dodając do niej ```cryptdevice=/dev/sda2:root```. Alternatywnie możemy zamiast tego dodać ```cryptdevice=UUID=<device-UUID>:root```, dzięki czemu zaszyfrowana partycja /dev/sda2 będzie rozpoznawana po UUID

## Uruchomienie systemu
Po dokończeniu instalacji, uruchamiamy system ponownie. Po wybraniu w bootloaderze odpowiedniego systemu otrzymamy komunikat, że musimy wprowadzić hasło do 
dostępu do partycji systemowej.
