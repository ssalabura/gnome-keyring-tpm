# Szyfrowanie dysku w Arch Linux

Arch umożliwia szyfrowanie "data at rest encryption". Polega to na tym, że każdy zapis/odczyt z dysku przechodzi przez dodatkową warstwę odpowiedzialną za szyfrowanie i deszyfrowanie danych. Ta warstwa może być nad warstwą systemu operacyjnego (stacked filesystem encryption - wtedy możliwe jest szyfrowanie tylko niektórych katalogów) lub pod (block device encryption - wtedy możliwe jest zaszyfrowanie np. całej partycji).

W ten sposób dane na dysku zostają zaszyfrowane podczas gdy dysk jest odmontowany (w szczególności, gdy komputer jest wyłączony). Zapobiega to atakom na dane podczas gdy np. komputer zostanie ukradziony lub gdy padnie dysk. Nie chroni ono jednak przed atakami na dysk, gdy atakujący ma dostęp do odblokowanego dysku (np. ataki poprzez internet na działający komputer) lub przed wyczyszczeniem danych z dysku przez atakującego.

Najpopularniejsze rozwiązanie stosowane w tym celu w Archu to dm-crypt, zawarty w _kernel-modules_, który umożliwia ponadto zaszyfrowanie prawie całego dysku (full disk encryption), łącznie z plikami systemu operacyjnego, swapem, plikami tymczasowymi i dziennikiem (/tmp, /var). Jedyne niezaszyfrowane pliki to pliki bootloadera i jądro systemu (zwykle). Takie szyfrowanie jest bezpieczniejsze, jednak wprowadza utrudnienie w postaci wymagania odszyfrowania dysku podczas rozruchu komputera, przed załadowaniem systemu. 

Proces szyfrowania systemu jest opisany tu: [Encrypting an entire system](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system) w kilku wariantach; w najprostszym po prostu szyfruje się osobno partycje inne niż te potrzebne do rozruchu, jednak nie daje to możliwości korzystania z partycji swap, wymaga utworzenia partycji wcześniej oraz umożliwia atakującemu podgląd na podział dysku.

Domyślnie szyfrowanie przez dm-crypt nie korzysta z Trusted Platform Module i na wiki Archa nie jest wspomniane, jak to wykonać, jednak podobno jest to możliwe [[link]](https://github.com/morbitzer/linux-luks-tpm-boot) 

## Inne alternatywy dla dm-crypt 
* TrueCrypt (i jego fork VeraCrypt), który działa również na systemach nielinuksowych, jednak nie wspiera on TPM
* Loop-AES, wymagający własnoręczej kompilacji innego jądra linuksowego 
* ZFS będący systemem plików
* eCryptfs 
* EncFS
* gocryptfs

przy czym ostatnie 3 działają nad warstwą systemu operacyjnego



Źródła:
* https://wiki.archlinux.org/index.php/Data-at-rest_encryption
* https://wiki.archlinux.org/index.php/Dm-crypt
* https://wiki.archlinux.org/index.php/TrueCrypt
