# Exercícios:
Atenção, a forma manual de gerenciamento de snapshots será usada para fins didáticos. É como aprender a multiplicação no papel para depois usar a calculadora!

## 1) Criar, formatar e montar um volume btrfs em arquivo de imagem no seu diretório de usuário.
```sh
mkdir testesSnapshot
cd testesSnapshot
truncate -s 200M imagem.img
mkfs.btrfs -L MeuTesteSnapshot imagem.img
mkdir imagemMontada
sudo mount -o loop imagem.img imagemMontada/
```
Conforme exercício 1 do Entendendo Subvolumes em BTRFS - Exercícios

## 2) Criar dois subvolumes: @arquivos e @snapshots
```sh
sudo btrfs subvolume create imagemMontada/@arquivos
sudo btrfs subvolume create imagemMontada/@snapshots
```
## 3) Setar o volume padrão da partição para o subvolume @arquivos
```
sudo btrfs subvol list ./imagemMontada
sudo btrfs subvol set-default XXX ./imagemMontada
```
## 4) Remontar com o novo volume padrão
```sh
sudo umount ./imagemMontada
sudo mount -o loop imagem.img imagemMontada/
```
## 5) Montar o subvolume @snapshots no diretório .snapshots
```sh
mkdir ./imagemMontada/.snapshots
sudo mount -o loop,subvol=@snapshots imagem.img imagemMontada/.snapshots
```
## 6) Criar um arquivo no diretório imagemMontada
```sh
sudo sh -c 'echo "Primeira versão do arquivo" > ./imagemMontada/exemplo.txt'
ls -la ./imagemMontada
```
## 7) Criar o primeiro snapshot do subvolume
```sh
sudo btrfs subvol snapshot -r ./imagemMontada ./imagemMontada/.snapshots/Snap1
```
Nota: É recomendável que os snapshots sejam somente-leitura (opção -r) para criar mais uma camada de proteção contra alterações inadvertidas

## 8) Sobrescrever o conteúdo do arquivo exemplo.txt

`sudo sh -c 'echo "Versão com conteúdo inesperado" > ./imagemMontada/exemplo.txt'`

## 9) Verificar o conteúdo do arquivo padrão e do arquivo que está no snapshot
```sh
cat ./imagemMontada/exemplo.txt
cat ./imagemMontada/.snapshots/Snap1/exemplo.txt
```
Nota: para esse simples exemplo com apenas um arquivo, seria muito mais fácil reverter apenas esse arquivo, copiando o arquivo do snapshot para o subvolume padrão. Porém em situações práticas, centenas de arquivos são adicionados, modificados ou deletados, tornando muito difícil comparar e reverter arquivo por arquivo manualmente (mais a frente veremos que o snapper automatiza esse processo também!)
## 10) Rollback #1: Criar um snapshot do snapshot a ser revertido em modo leitura e gravação. Confirme os modos
```sh
sudo btrfs subv snap ./imagemMontada/.snapshots/Snap1 ./imagemMontada/.snapshots/SnapRW
sudo btrfs property get ./imagemMontada/.snapshots/Snap1
sudo btrfs property get ./imagemMontada/.snapshots/SnapRW
```
## 11) Rollback #2: Montar a árvore do BTRFS para realizar operações de movimentação de subvolumes
```sh
mkdir btrfs
sudo mount -o loop,subvolid=5 imagem.img ./btrfs/
```
## 12) Rollback #3: Renomear o subvolume @arquivos para @arquivos.old
`sudo mv ./btrfs/@arquivos ./btrfs/@arquivos.old`
Nota: o subvolume continua montado em imagemMontada, mas não houve erro nessa operação.
## 13) Rollback #4: Renomear o SnapRW para @arquivos, movendo-o de subvolume pai
```sh
sudo btrfs subv list ./btrfs
sudo mv ./btrfs/@snapshots/SnapRW/ ./btrfs/@arquivos
```
Nota: É importante que o subvolume seja renomeado no caso de haver referências a ele durante a inicialização (caso possua a pasta /boot) ou no /etc/fstab. Caso ele seja apenas montado como subvolume padrão em outros pontos de montagem, essa etapa de renomear o subvolume é opcional, porém recomendável. Fica um tanto estranho ter o volume padrão dentro do subvolume @snapshots e não poder deletá-lo.
## 14) Rollback #5: Alterar o volume padrão do sistema de arquivos para o número do novo @arquivos
```sh
sudo btrfs subv list ./btrfs
sudo btrfs subv set-default XXX ./btrfs
```
## 15) Desmonte tudo e remonte o subvolume padrão e seus snapshots (simulando o reboot do computador)
```sh
sudo umount -R ./imagemMontada
sudo umount ./btrfs
sudo mount -o loop imagem.img imagemMontada/
sudo mount -o loop,subvol=@snapshots imagem.img imagemMontada/.snapshots
```
Nota: a opção -R do umount desmonta todos os demais pontos de montagem recursivamente dentro do ponto de montagem especificado.

## 16) Compare o conteúdo dos arquivos exemplo do subvolume padrão com o do snapshot
```sh
cat ./imagemMontada/exemplo.txt
cat ./imagemMontada/.snapshots/Snap1/exemplo.txt
```
## 17) Uma vez que a operação foi completada com sucesso, poderá apagar o subvolume indesejado.
```sh
sudo mount -o loop,subvolid=5 imagem.img ./btrfs/
sudo btrfs subvol delete ./btrfs/@arquivos.old
```

# Exercícios solucionados:

<pre><font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ # 1
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ mkdir testesSnapshot
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~</b></font>$ cd testesSnapshot
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ truncate -s 200M imagem.img
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ mkfs.btrfs -L MeuTesteSnapshot imagem.img
btrfs-progs v4.15.1
See http://btrfs.wiki.kernel.org for more information.

Label:              MeuTesteSnapshot
UUID:               548770cc-7302-453d-a28a-82a0052e2278
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

<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ mkdir imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop imagem.img imagemMontada/
[sudo] senha para john:     
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 2
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvolume create imagemMontada/@arquivos
Create subvolume &apos;imagemMontada/@arquivos&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvolume create imagemMontada/@snapshots
Create subvolume &apos;imagemMontada/@snapshots&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 3
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvol list ./imagemMontada
ID 256 gen 8 top level 5 path @arquivos
ID 257 gen 9 top level 5 path @snapshots
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvol set-default 256 ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 4
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo umount ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop imagem.img imagemMontada/
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 5
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mkdir ./imagemMontada/.snapshots
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop,subvol=@snapshots imagem.img imagemMontada/.snapshots
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 6
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo sh -c &apos;echo &quot;Primeira versão do arquivo&quot; &gt; ./imagemMontada/exemplo.txt&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ ls -la ./imagemMontada
total 4
drwxr-xr-x 1 root root 42 mar 17 22:38 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 1 john john 46 mar 17 22:33 <font color="#729FCF"><b>..</b></font>
-rw-r--r-- 1 root root 28 mar 17 22:38 exemplo.txt
drwxr-xr-x 1 root root  0 mar 17 22:33 <font color="#729FCF"><b>.snapshots</b></font>
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 7
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvol snapshot -r ./imagemMontada ./imagemMontada/.snapshots/Snap1
Create a readonly snapshot of &apos;./imagemMontada&apos; in &apos;./imagemMontada/.snapshots/Snap1&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 8
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo sh -c &apos;echo &quot;Versão com conteúdo inesperado&quot; &gt; ./imagemMontada/exemplo.txt&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 9
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ cat ./imagemMontada/exemplo.txt
Versão com conteúdo inesperado
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ cat ./imagemMontada/.snapshots/Snap1/exemplo.txt
Primeira versão do arquivo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 10
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subv snap ./imagemMontada/.snapshots/Snap1 ./imagemMontada/.snapshots/SnapRW
Create a snapshot of &apos;./imagemMontada/.snapshots/Snap1&apos; in &apos;./imagemMontada/.snapshots/SnapRW&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs property get ./imagemMontada/.snapshots/Snap1
ro=true
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs property get ./imagemMontada/.snapshots/SnapRW
ro=false
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 11
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ mkdir btrfs
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop,subvolid=5 imagem.img ./btrfs/
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 12
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mv ./btrfs/@arquivos ./btrfs/@arquivos.old
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 13
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subv list ./btrfs
ID 256 gen 16 top level 5 path @arquivos.old
ID 257 gen 17 top level 5 path @snapshots
ID 258 gen 17 top level 257 path @snapshots/Snap1
ID 259 gen 17 top level 257 path @snapshots/SnapRW
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mv ./btrfs/@snapshots/SnapRW/ ./btrfs/@arquivos
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 14
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subv list ./btrfs
ID 256 gen 16 top level 5 path @arquivos.old
ID 257 gen 19 top level 5 path @snapshots
ID 258 gen 17 top level 257 path @snapshots/Snap1
ID 259 gen 17 top level 5 path @arquivos
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subv set-default 259 ./btrfs
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 15
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo umount -R ./imagemMontada
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo umount ./btrfs
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop imagem.img imagemMontada/
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop,subvol=@snapshots imagem.img imagemMontada/.snapshots
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 16
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ cat ./imagemMontada/exemplo.txt
Primeira versão do arquivo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ cat ./imagemMontada/.snapshots/Snap1/exemplo.txt
Primeira versão do arquivo
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ # 17
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo mount -o loop,subvolid=5 imagem.img ./btrfs/
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ sudo btrfs subvol delete ./btrfs/@arquivos.old
Delete subvolume (no-commit): &apos;/home/john/testesSnapshot/btrfs/@arquivos.old&apos;
<font color="#8AE234"><b>john@MintVM</b></font>:<font color="#729FCF"><b>~/testesSnapshot</b></font>$ 
</pre>
