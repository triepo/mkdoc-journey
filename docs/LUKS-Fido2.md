
On this page I will explain, how to decrypt LUKS encrypted partition

### Prerequisites
 * Debian 13 setup with encryption
 * Hardware fido2 Token (e.g.: yubikey, Token2)

### Find Partition
 First we need find to find the encrypted filesystems.
 We will use `lsblk` to figure that out

```shell title="Output of 'lsblk'" linenums="1" hl_lines="7-10"
nimda@trixie:~$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sr0                      11:0    1  3,7G  0 rom   
vda                     254:0    0   20G  0 disk  
├─vda1                  254:1    0  966M  0 part  /boot
├─vda2                  254:2    0    1K  0 part  
└─vda5                  254:5    0 19,1G  0 part  
  └─vda5_crypt          253:0    0   19G  0 crypt 
    ├─trixie--vg-root   253:1    0   18G  0 lvm   /
    └─trixie--vg-swap_1 253:2    0    1G  0 lvm   [SWAP]
nimda@trixie:~$ 
```
In this case we have `/dev/vda5`

### Show fido2 key
Next action will be, to check, if fido2 token is found using `systemd-cryptenroll`command

```shell title="Output of 'systemd-cryptenroll'"
nimda@trixie:~$ systemd-cryptenroll --fido2-device=list
PATH         MANUFACTURER PRODUCT                  COMPATIBLE
/dev/hidraw2 TOKEN2       FIDO2 Security Key(0026) ✓
nimda@trixie:~$ 
```
Now, as we have all needed information, we can enroll the token

### Enroll
```shell title="Enrolling token"
nimda@trixie:~$ sudo systemd-cryptenroll /dev/vda5 --fido2-device=auto -fido2-with-client-pin=yes
[sudo] Passwort für nimda: 
🔐 Please enter current passphrase for disk /dev/vda5: •••••••••               
Initializing FIDO2 credential on security token.
👆 (Hint: This might require confirmation of user presence on security token.)
🔐 Please enter security token PIN: ••••••                  
Generating secret key on FIDO2 security token.
👆 In order to allow secret key generation, please confirm presence on security token.
New FIDO2 token enrolled as key slot 1.
nimda@trixie:~$ 
```

### Checking
To check, that the token was configured correctly, we can use the command `sudo cryptsetup luksDump /dev/vda5`

```shell title="Output of 'cryptsetup luksDump /dev/vda5'"
~~~
Tokens:
  0: systemd-fido2
        fido2-credential:
                    98 9b 58 d4 52 08 4f 1b 65 2a e9 c7 86 6b 45 ba
                    66 3b c1 7f cb d4 39 0b e4 5e 3b 4b ea 36 5d 0e
                    7c b8 50 38 a2 5a 31 09 d7 91 1e cd 78 46 4c cb
                    fc 6f 51 ee da b5 61 79 44 07 82 2a 56 92 ad bf
                    2d cb cc 53 1b c6 70 8b 57 4b 9f 75 59 60 be 54
                    21 cf 89 b0 f5 1b 28 8e 2f 3c 5d 44 f5 5e 7c da
        fido2-salt: 4a f5 09 cf a8 84 66 f5 9e e4 3e 6e c3 19 1a a1
                    7e 52 d5 e3 ef cd 45 bd e3 60 02 09 94 45 ea 00
        fido2-rp:   io.systemd.cryptsetup
        fido2-clientPin-required:
                    true
        fido2-up-required:
                    true
        fido2-uv-required:
                    false
        Keyslot:    1
~~~
```
### Edit /etc/crypttab
Next step is to add `,fido2-device=auto` to file `/etc/crypttab`

```shell title="Content of '/etc/crypttab'"
nimda@trixie:~$ cat /etc/crypttab 
vda5_crypt UUID=fb3239d3-be24-446e-b956-8fe4aea8f34c none luks,discard,x-initrd.attach,fido2-device=auto
nimda@trixie:~$
```

### initramfs
As the tool `initramfs` has a bug not recognizing the option we added, we need to install `dracut`

## dracut
Create config file for `dracut`
```shell title="'dracut' config"
sudo mkdir /etc/dracut.conf.d
echo 'hostonly="yes"' | sudo tee /etc/dracut.conf.d/hostonly.conf
```

```shell title="Install 'dracut'"
sudo apt install dracut
```

After installation we need to recreate the 'initramfs'
```shell title="Recreate 'initramfs'"
sudo dracut -f
```
