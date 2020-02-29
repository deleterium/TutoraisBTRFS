# Exercícios
## 1) Criar, formatar e montar um volume btrfs em arquivo de imagem no seu diretório de usuário.

```js
mkdir testesBTRFS
cd testesBTRFS
truncate -s 200M imagem.img
mkfs.btrfs -L MinhaImagemBTRFS imagem.img
mkdir imagemMontada
sudo mount -o loop imagem.img imagemMontada/
```
Notas: O comando `truncate` somente aloca espaço no disco, portanto o conteúdo do arquivo é desconhecido. O `mkfs.btrfs` faz a formatação desse espaço reservado para organizar o sistema de arquivos e gravar a informação de espaço livre. Se não há esse comando instale o pacote **btrfs-progs** ou **btrfs-tools** (depende da distribuição). A opção **-o loop** no comando `mount` serve para montar arquivos como se fossem dispositivos de bloco.

## 2) Criar dois subvolumes, @raiz e @home.

```js
sudo btrfs subvolume create imagemMontada/@raiz
sudo btrfs subvolu crea imagemMontada/@home
```
Nota: As opções do comando `btrfs` podem ser abreviadas desde que não causem confusão com outras opções.
## 3) Verificar os ID’s dos subvolumes criados
`sudo btrfs subvol list ./imagemMontada`

## 4) Qual o ID padrão do volume btrfs?
`sudo btrfs subv get-default ./imagemMontada`
## 5) Crie um diretório e copie um arquivo para dentro do imagemMontada:
```sh
mkdir ./imagemMontada/novodir
cp /etc/fstab ./imagemMontada/novodir
```
Nota: Saber trabalhar com permissões de arquivos é pré-requisito para esse tutorial. Caso não saiba veja a cola: `sudo chown SeuUsuario imagemMontada/.`
## 6) Liste o conteúdo da imagemMontada:
`ls -la ./imagemMontada`
Nota: Além do diretório criado também há "diretórios" que são subvolumes!
## 7) Copie um arquivo para um subvolume sem precisar montá-lo especificamente
```sh
sudo cp /etc/profile ./imagemMontada/@raiz
ls -la ./imagemMontada/@raiz
```
Nota: O autocompletar não funciona muito bem com o @, portanto as vezes tem que escrever tudo…
## 8) Monte um subvolume:
```sh
mkdir subvolumeMontado
sudo mount -o loop,subvol=@raiz imagem.img ./subvolumeMontado
```
Nota: A opção **loop** só e necessária por causa do arquivo ser imagem. Como o arquivo imagem.img já está montado, é possível também montar usando o dispositivo de bloco já configurado. O comando seria `sudo mount -o subvol=@raiz /dev/loop0 ./subvolumeMontado` . 

## 9) Liste o conteúdo do subvolume Montado
`ls -la ./subvolumeMontado`

Nota: Veja que já está lá o arquivo que foi copiado via Árvore!
## 10) Altere o id-padrão do volume para o subvolume @home
```sh
sudo btrfs sub li ./imagemMontada
sudo btrfs subvolume set-default XXX ./imagemMontada
```
Nota: Substitua o valor XXX pelo ID mostrado pelo **list**.
## 11) Liste o id-padrão do subvolume @raiz
`sudo btrfs subvolume get-default ./subvolumeMontado`

Nota: Ao se alterar o ID-padrão de qualquer subvolume, será alterado o ID-padrão do volume como um todo. Da mesma forma pegar a variável id-padrão de qualquer subvolume retornará o valor do id-padrão da Árvore.

## 12) Desmonte tudo
```sh
sudo umount ./imagemMontada
sudo umount ./subvolumeMontado
```
## 13) Monte a imagem sem especificar subvolume e liste seu conteúdo
```sh
sudo mount -o loop imagem.img ./imagemMontada
ls -la ./imagemMontada
```
Nota: Como era de se esperar, foi montado o subvolume @home, que está vazio.

## 14) Liste os subvolumes do diretório imagemMontada
```sh
sudo btrfs subvo lis ./imagemMontada
```
Nota: Foram mostrados todos os subvolumes da Árvore. Talvez alguns esperassem que fosse mostrado apenas os subvolumes do subvolume @home, mas não é assim que funciona. O **list** sempre mostra todos os subvolumes!

## 15) Crie um arquivo e um subvolume no subvolume @home
```sh
sudo touch ./imagemMontada/arquivo.txt
sudo btrfs subv creat ./imagemMontada/temp-files
```
## 16) Liste o conteúdo do diretório imagemMontada

`ls -la ./imagemMontada`

Nota: Essa é uma estratégia que pode ser usada para snapshots, onde os subvolumes não são monitorados. Neste caso um futuro snapshot do @home não conterá os arquivos dentro do temp-files. 

## 17) Crie mais um subvolume na Árvore, nomeado @tmp.

Nota: a árvore não está montada, então primeiro teremos que montá-la para depois criar o subvolume. Não é possível acessá-la pelo ponto de montagem imagemMontada, pois ele está montado como um subvolume e não é possível acessar volumes de hieraquia anterior a ele.
```sh
mkdir arvoreMontada
sudo mount -o loop,subvolid=5 imagem.img ./arvoreMontada
sudo btrfs subvol create ./arvoreMontada/@tmp
sudo btrfs sub list ./arvoreMontada
```
Nota: Número mágico 5 aparecendo aqui para montar a Árvore, pois não é possível montá-la pelo nome. Note também que com a árvore montada é possível criar subvolumes em qualquer subvolume! Outro ponto é perceber o relacionamento dos subvolumes, com o **top level** indicando o subvolume pai. Neste caso apenas o **temp-files** não é filho direto da árvore, mas sim do subvolume @home (perceba pelo id do top level)

## 18) Renomeie o subvolume @raiz para @rootfs

Nota: Não existe um comando específico do btrfs para renomear o volume. Porém mover o subvolume como se fosse um diretório resolve a situação. Para essas operações monte a Árvore. Mover um subvolume não altera seu ID, então se ele for o ID-padrão, continuará sendo.
```sh
mv ./arvoreMontada/@raiz ./arvoreMontada/@rootfs
sudo btrfs sub list ./arvoreMontada
```
## 19) Mova o subvolume @tmp para dentro do subvolume @home, renomeando-o para tmp
```sh
sudo mv ./arvoreMontada/@tmp ./arvoreMontada/@home
sudo mv ./arvoreMontada/@home/@tmp ./arvoreMontada/@home/tmp
sudo btrfs sub list ./arvoreMontada
```
Nota: Mover um subvolume para dentro de outro subvolume não altera seu ID, mas altera o subvolume pai. Veja como o **top level** do subvolume **tmp** mudou.
## 20) Monte o subvolume padrão e crie um arquivo de 120MB
```sh
sudo mount -o loop imagem.img ./imagemMontada
truncate -s 120M ./imagemMontada/arquivogrande.lixo
ls -la ./imagemMontada/arquivogrande.lixo
```
## 21) Crie uma cópia desse arquivo no subvolume temp-files.
`cp ./imagemMontada/arquivogrande.lixo ./imagemMontada/tmp-files`

Nota: Como era de se esperar, não há espaço suficiente no disco para dois arquivos de 120M, a imagem foi criada com 200MB. Porém uma das vantagens do btrfs é que ele não precisa copiar o conteúdo, ele pode apenas indicar que o mesmo conteúdo está em outro lugar do disco. Para tal usamos a opção **--reflink=always** ao copiar.

`cp --reflink=always ./imagemMontada/arquivogrande.lixo ./imagemMontada/tmp-files`

A cópia foi efetuada! Esse é o princípio de funcionamento dos snapshots, onde uma grande quantidade de arquivos parece estar duplicada, mas guardam suas referências de conteúdo. 

## 22) Remova o arquivo do volume padrão e verifique o espaço livre:
```sh
rm ./imagemMontada/arquivogrande.lixo
df ./imagemMontada
```
Nota: O espaço livre não foi liberado pois o conteúdo ainda é referenciado em outro local. 
## 23) Remova o arquivo do subvolume e verifique o espaço livre:
```sh
rm ./imagemMontada/tmp-files/arquivogrande.lixo
df ./imagemMontada
```
Nota: Agora sim o espaço foi liberado. Lembre-se que o espaço só será liberado quando não houver nenhuma referência ao conteúdo do arquivo. Isso pode ser fácil quando há poucos subvolumes, mas quando há 50 snapshots fica difícil. Nessa hora que os programas que auxiliam essa tarefa entram em jogo, como o `snapper`. Mas isso é assunto para um próximo tutorial!

# Exercícios solucionados


<pre><font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ # 1
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ mkdir testesBTRFS
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ cd testesBTRFS
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ truncate -s 200M imagem.img
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkfs.btrfs -L MinhaImagemBTRFS imagem.img
btrfs-progs v4.15.1
See http://btrfs.wiki.kernel.org for more information.

Label:              MinhaImagemBTRFS
UUID:               0e507c12-de93-4d14-be5a-c361660af3ce
Node size:          16384
Sector size:        4096
Filesystem size:    200.00MiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP              32.00MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1   200.00MiB  imagem.img

<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkdir imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mount -o loop imagem.img imagemMontada/
[sudo] senha para john:     
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 2
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvolume create imagemMontada/@raiz
Create subvolume &apos;imagemMontada/@raiz&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvolu crea imagemMontada/@home
Create subvolume &apos;imagemMontada/@home&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 3
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs sub list ./arvoreMontada
ID 256 gen 18 top level 5 path @raiz
ID 257 gen 25 top level 5 path @home
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 4
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvolume get-default ./imagemMontada
ID 5 (FS_TREE)
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 5
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkdir ./imagemMontada/novodir
mkdir: não foi possível criar o diretório “./imagemMontada/novodir”: Permissão negada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada/
total 16
drwxr-xr-x 1 root root 20 fev 29 10:56 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john 46 fev 29 10:56 <font color="#729FCF"><b>..</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@home</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@raiz</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo chown john:john imagemMontada/.
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada/
total 16
drwxr-xr-x 1 john john 20 fev 29 10:56 <span style="background-color:#4E9A06"><font color="#3465A4">.</font></span>
drwxr-xr-x 1 john john 46 fev 29 10:56 <font color="#729FCF"><b>..</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@home</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@raiz</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkdir ./imagemMontada/novodir
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ cp /etc/fstab ./imagemMontada/novodir
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 6
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada/
total 16
drwxr-xr-x 1 john john 34 fev 29 11:04 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john 46 fev 29 10:56 <font color="#729FCF"><b>..</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@home</b></font>
drwxr-xr-x 1 john john  0 fev 29 11:04 <font color="#729FCF"><b>novodir</b></font>
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>@raiz</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 7
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo cp /etc/profile ./imagemMontada/@raiz
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada/@raiz
total 20
drwxr-xr-x 1 root root  14 fev 29 11:16 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john  34 fev 29 11:04 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 581 fev 29 11:16 profile
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 8
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkdir subvolumeMontado
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mount -o loop,subvol=@raiz imagem.img ./subvolumeMontado
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 9
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./subvolumeMontado
total 8
drwxr-xr-x 1 root root  24 fev 29 11:17 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john  78 fev 29 11:22 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 581 fev 29 11:16 profile
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 10
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs sub li ./imagemMontada
ID 256 gen 18 top level 5 path @raiz
ID 257 gen 10 top level 5 path @home
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvolume set-default 257 ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 11
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvolume get-default ./subvolumeMontado
ID 257 gen 10 top level 5 path @home
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ 
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 12
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo umount ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo umount ./subvolumeMontado
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 13
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mount -o loop imagem.img ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada
total 0
drwxr-xr-x 1 root root  0 fev 29 10:59 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john 78 fev 29 11:22 <font color="#729FCF"><b>..</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 14
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvo lis ./imagemMontada
ID 256 gen 18 top level 5 path @raiz
ID 257 gen 10 top level 5 path @home
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 15
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo touch ./imagemMontada/arquivo.txt
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subv creat ./imagemMontada/temp-files
Create subvolume &apos;./imagemMontada/temp-files&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 16
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada
total 0
drwxr-xr-x 1 root root 42 fev 29 11:51 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john 78 fev 29 11:22 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root  0 fev 29 11:51 arquivo.txt
drwxr-xr-x 1 root root  0 fev 29 11:51 <font color="#729FCF"><b>temp-files</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 17
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mkdir arvoreMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mount -o loop,subvolid=5 imagem.img ./arvoreMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs subvol create ./arvoreMontada/@tmp
Create subvolume &apos;./arvoreMontada/@tmp&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs sub list ./arvoreMontada
ID 256 gen 18 top level 5 path @raiz
ID 257 gen 25 top level 5 path @home
ID 258 gen 24 top level 257 path @home/temp-files
ID 259 gen 26 top level 5 path @tmp
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 18
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ mv ./arvoreMontada/@raiz ./arvoreMontada/@rootfs
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs sub list ./arvoreMontada
ID 256 gen 18 top level 5 path @rootfs
ID 257 gen 25 top level 5 path @home
ID 258 gen 24 top level 257 path @home/temp-files
ID 259 gen 26 top level 5 path @tmp
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ 
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 19
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mv ./arvoreMontada/@tmp ./arvoreMontada/@home
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo mv ./arvoreMontada/@home/@tmp ./arvoreMontada/@home/tmp
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs sub list ./arvoreMontada
ID 256 gen 28 top level 5 path @rootfs
ID 257 gen 28 top level 5 path @home
ID 258 gen 28 top level 257 path @home/temp-files
ID 259 gen 28 top level 257 path @home/tmp
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 20
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo dd if=/dev/urandom of=./imagemMontada/arquivogrande.lixo bs=1M count=100
100+0 registros de entrada
100+0 registros de saída
104857600 bytes (105 MB, 100 MiB) copiados, 1,756 s, 59,7 MB/s
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -la ./imagemMontada
total 102400
drwxr-xr-x 1 root root        62 fev 29 13:41 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john       104 fev 29 11:55 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 104857600 fev 29 13:41 arquivogrande.lixo
drwxr-xr-x 1 root root         0 fev 29 13:34 <font color="#729FCF"><b>temp-files</b></font>
drwxr-xr-x 1 root root         0 fev 29 13:34 <font color="#729FCF"><b>tmp</b></font>

<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 21
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo cp ./imagemMontada/arquivogrande.lixo ./imagemMontada/tmp
cp: erro de escrita de &apos;./imagemMontada/tmp/arquivogrande.lixo&apos;: Não há espaço disponível no dispositivo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo cp --reflink=always ./imagemMontada/arquivogrande.lixo ./imagemMontada/tmp
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 22
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ ls -laR ./imagemMontada/
./imagemMontada/:
total 204800
drwxr-xr-x 1 root root        80 fev 29 13:43 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john       104 fev 29 11:55 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 104857600 fev 29 13:41 arquivogrande.lixo
drwxr-xr-x 1 root root         0 fev 29 13:56 <font color="#729FCF"><b>temp-files</b></font>
drwxr-xr-x 1 root root        36 fev 29 13:57 <font color="#729FCF"><b>tmp</b></font>

./imagemMontada/temp-files:
total 0
drwxr-xr-x 1 root root  0 fev 29 13:56 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 root root 80 fev 29 13:43 <font color="#729FCF"><b>..</b></font>

./imagemMontada/tmp:
total 102400
drwxr-xr-x 1 root root        36 fev 29 13:57 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 root root        80 fev 29 13:43 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 104857600 fev 29 13:58 arquivogrande.lixo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 23
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs filesystem usage ./imagemMontada
Overall:
    Device size:		 200.00MiB
    Device allocated:		 188.00MiB
    Device unallocated:		  12.00MiB
    Device missing:		     0.00B
    Used:			 100.59MiB
    Free (estimated):		   8.00MiB	(min: 8.00MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,single: Size:108.00MiB, Used:100.00MiB
   /dev/loop0	 108.00MiB

Metadata,DUP: Size:32.00MiB, Used:288.00KiB
   /dev/loop0	  64.00MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/loop0	  16.00MiB

Unallocated:
   /dev/loop0	  12.00MiB
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs filesystem du ./imagemMontada
     Total   Exclusive  Set shared  Filename
     0.00B       0.00B           -  ./imagemMontada/temp-files
 100.00MiB       0.00B           -  ./imagemMontada/tmp/arquivogrande.lixo
 100.00MiB       0.00B           -  ./imagemMontada/tmp
 100.00MiB       0.00B           -  ./imagemMontada/arquivogrande.lixo
 200.00MiB       0.00B   100.00MiB  ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 24
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo rm ./imagemMontada/arquivogrande.lixo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs filesystem sync ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs file us ./imagemMontada
Overall:
    Device size:		 200.00MiB
    Device allocated:		 188.00MiB
    Device unallocated:		  12.00MiB
    Device missing:		     0.00B
    Used:			 100.59MiB
    Free (estimated):		   8.00MiB	(min: 8.00MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,single: Size:108.00MiB, Used:100.00MiB
   /dev/loop0	 108.00MiB

Metadata,DUP: Size:32.00MiB, Used:288.00KiB
   /dev/loop0	  64.00MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/loop0	  16.00MiB

Unallocated:
   /dev/loop0	  12.00MiB
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ # 25
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo rm ./imagemMontada/tmp/arquivogrande.lixo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs filesystem sync ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ sudo btrfs filesystem usage ./imagemMontada
Overall:
    Device size:		 200.00MiB
    Device allocated:		  92.00MiB
    Device unallocated:		 108.00MiB
    Device missing:		     0.00B
    Used:			 384.00KiB
    Free (estimated):		 120.00MiB	(min: 66.00MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 32.00KiB)

Data,single: Size:12.00MiB, Used:0.00B
   /dev/loop0	  12.00MiB

Metadata,DUP: Size:32.00MiB, Used:176.00KiB
   /dev/loop0	  64.00MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/loop0	  16.00MiB

Unallocated:
   /dev/loop0	 108.00MiB
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesBTRFS</b></font>$ 
</pre>
