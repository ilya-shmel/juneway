Tags: # - [[DevOps]] - [[Juneway]]

Related: [[2023-12-04]]

Source: [Утилита времен «динозавров»](https://habr.com/ru/companies/selectel/articles/655485/)

--- 
## Интерпретатор `zsh`
Создаю каталоги для `chroot`
```bash 
cd ~/Desktop/juneway
mkdir Lesson3
mkdir bin
```

Копирую утилиту `zsh` и ее зависимые библиотеки
```bash
sudo cp /bin/zsh Lesson3/bin

ldd Lesson3/bin/zsh 
	linux-vdso.so.1 (0x00007fff574d5000)
	libtinfo.so.6 => /lib64/libtinfo.so.6 (0x00007f988596b000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f988588a000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f98856ac000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f9885abb000)

cp /lib64/libtinfo.so.6 lib64 
➜  Lesson3 cp /lib64/libm.so.6 lib64 
➜  Lesson3 cp /lib64/libc.so.6 lib64
➜  Lesson3 cp /lib64/ld-linux-x86-64.so.2 lib64
➜  Lesson3 ll lib64
```

Запускаю `zsh`
```bash
sudo chroot Lesson3 
zsh: failed to load module `zsh/complete': /usr/lib64/zsh/5.9/zsh/complete.so: cannot open shared object file: No such file or directory
```

Оболочка запустилась, но с ошибками. Копирую специфические библиотеки `zsh` и пробую снова
```bash
mkdir usr/lib64/zsh/5.9/zsh
cp /usr/lib64/zsh/5.9/zsh ~/Desktop/juneway/Lesson3/usr/lib64/zsh/5.9/zsh

sudo chroot Lesson3
lian-li#
lian-li# exit
```
На этот раз все работает.
## Запуск `cp`

Копирую зависимые библиотеки
```bash 
cp /lib64/libselinux.so.1 ~/Desktop/juneway/Lesson3/lib64 
cp /lib64/libacl.so.1 ~/Desktop/juneway/Lesson3/lib64
cp /lib64/libattr.so.1 ~/Desktop/juneway/Lesson3/lib64
cp /lib64/libc.so.6 ~/Desktop/juneway/Lesson3/lib64
cp /lib64/libpcre2-8.so.0 ~/Desktop/juneway/Lesson3/lib64
cp /lib64/ld-linux-x86-64.so.2 ~/Desktop/juneway/Lesson3/lib64
```

Пробую запустить 
```bash 
lian-li# cp
cp: missing file operand
Try 'cp --help' for more information.
```

Ключ `--help` работает
```bash 
lian-li# cp --help                                                          
Usage: cp [OPTION]... [-T] SOURCE DEST
  or:  cp [OPTION]... SOURCE... DIRECTORY
  or:  cp [OPTION]... -t DIRECTORY SOURCE...
Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.
........................................................
GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
Report any translation bugs to <https://translationproject.org/team/>
Full documentation <https://www.gnu.org/software/coreutils/cp>
or available locally via: info '(coreutils) cp invocation'
```

Создаю файлы для тестирования утилиты
```bash 
➜  Lesson3 mkdir -p home/Desktop
➜  Lesson3 mkdir -p home/Downloads
➜  Lesson3 echo "The first file." > home/Desktop/file1.txt
➜  Lesson3 echo "The second file." > home/Downloads/file2.txt
```

Копирую второй файл из `/home/Downloads` в `/home/Desktop` (почему слэши двойные???)
```bash 
sudo chroot Lesson3
lian-li~ cp /home//Downloads//file2.txt  /home//Desktop/
```

Пытаюсь проверить результат
```bash
lian-li# ll home/Desktop                                   
zsh: command not found: ll
```

Для этого нужно установить утилиту `ls`.

## Установка `ls`

Копирую исполняемый файл и проверяю зависимости
```bash 
cp /bin/ls ~/Desktop/juneway/Lesson3/bin 

ldd bin/ls
	linux-vdso.so.1 (0x00007fff99b01000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f5866fba000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007f5866fb0000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f5866dd2000)
	libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f5866d38000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f586702c000)
```

Большая часть библиотек уже скопирована, кроме одной
```bash 
➜  Lesson3 cp /lib64/libcap.so.2 lib64
```

Проверяю работоспособность утилиты
```bash 
lian-li# ls
bin  home  lib	lib64  usr
```

Проверяю результат работы `cp`. Файл скопирован
```bash 
lian-li# ls home/Desktop/ 
file1.txt  file2.txt
```
