## An example of codeblock in python

```py title="Add numbers.py"
# Function to add two numbers
def add_two_numbers(num1, num2):
    return num1 + num2

# Example usage
result = add_two_numbers(5, 3)
print('The sum is:', result)
```

```shell title="Test"
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

```
