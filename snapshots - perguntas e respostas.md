# Entendendo snapshots em BTRFS

O snapshot, ou instantâneo como traduzido em alguns programas, registra um estado instantâneo do sistema de arquivos em um subvolume, de forma análoga ao modo que uma fotografia captura um instante na paisagem. Para tal feito o sistema de arquivos já deve ser criado com essa característica desde a sua concepção. Por isso não podemos obter snapshots de sistemas ext4 ou xfs.

Para entender na totalidade os snapshots em sistemas BTRFS, é necessário que já se tenha compreensão do funcionamento de subvolumes. Leia, compreenda e exercite o texto anterior: Entendendo subvolumes em BTRFS

## 1) Como o snapshot funciona?
Quando é dado o comando para gerar um instantaneo, todos os acessos ao disco são temporariamente suspensos. O comando então cria um novo subvolume e replica a informação dos metadados do subvolume especificado para este novo subvolume de snapshot. Por padrão este subvolume é apenas leitura. Uma vez finalizada a operação, as operações ao disco são retomadas.

## 2) Como que um snapshot não aumenta o espaço ocupado em disco?
Os arquivos são “copiados” apenas por referência. Os dados em disco continuam no mesmo local, apenas é criada uma nova referência para ele. No caso do snapshot TODOS os arquivos são duplicados por referência. 

## 3) Mas o que acontece quando um arquivo é modificado?
A informação de um arquivo modificado em um subvolume passa a ocupar um novo espaço físico. Se há snapshots desse arquivo, o conteúdo antigo é preservado. Os dados só serão descartados quando não houver mais referência a eles em todo o disco. É por isso que os snapshots começam ocupando bem pouco espaço (apenas mais informação em metadados) e conforme há alterações nos arquivos do subvolume principal, o sistema indica que há mais espaço ocupado pelos subvolumes.

## 4) Como organizar os snapshots?
Há duas abordagens comuns para organização dos subvolumes:
1) Realizar os snapshots em um subdiretório oculto de nome .snapshots facilita o processo de criação de snapshots porém dificulta o rollback. Preferível usar em subvolumes que não precisarão de rollbacks, mas que seja interessante manter histórico de alterações de arquivos (timeline).
2) Criar um subvolume @snapshots e então montá-lo na inicialização em um diretório .snapshots é mais trabalhoso para configurar, mas facilita em um possível rollback. Preferível nos casos onde o rollback pode ser frequente.
Nota: o processo de recuperação de arquivos isolados é o mesmo, independente do método escolhido. Como esse tutorial está mais focado para a reversão do sistema, será usada a segunda opção.

## 5) Como que eu reverto um snapshot? Ou ainda, como fazer o rollback de um snapshot?
Isso já vai depender do modo com que o snapshot foi feito e como é a distribuição de subvolumes no disco. De uma forma geral, para não perder a rastreabilidade de arquivos, costuma-se criar um snapshot do snapshot desejado. Em seguida altera-se esse snapshot para habilitar leitura e gravação. Então torna-se esse subvolume como padrão e remonta-se o subvolume (ou reinicia-se o computador no caso do raiz). Terá novamente os arquivos como naquele momento.

Esse procedimento pode ter vários complicadores, principalmente se for o raiz de uma instalação Linux, e se ele contém os arquivos de inicialização (pasta /boot). Para facilitar essa sequência de ações, o programa snapper foi criado e será alvo de um futuro tutorial.

## 6) Quero optar pelo gerenciamento manual. Quais recomendações?
Manter muitos snapshots (mais de 20) pode impactar de forma sensível a performance do disco. O espaço ocupado pelos snapshots vai crescendo conforme há mais modificações no sistema de arquivos, facilmente levando fim do espaço disponível. Evite manter snapshots muito antigos e se possível gerencie o espaço ocupado pelos snapshots com o comando `btrfs qgroup show <dir>`. Verifique, instale e configure o pacote btrfsmaintenance, que mantém rotinas para balancear os sistemas.
