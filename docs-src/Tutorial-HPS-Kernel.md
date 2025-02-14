# Tutorial 7 - HPS - Compilando o kernel

<iframe width="560" height="315" src="https://www.youtube.com/embed/SOXeXauRAm0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---------------------

<iframe width="560" height="315" src="https://www.youtube.com/embed/FEfSlk1EHNI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Nesse tutorial iremos compilar o kernel do Linux para o ARM do nosso SoC usando o toolchain que já temos configurado.

## Kernel 4.4

Clone o kernel do linux :

``` bash
$ git clone https://git.kernel.org
$ cd linux
```

Vamos trabalhar com a versão `4.4` do kernel que é uma versão com: `Long Time Suport` (LTS), ou seja, será mantida por muito mais tempo que as outras versões. A versão 4.4 foi lançada em 10 de Janeiro e será mantida oficialmente até 2021, ela é também a versão Super LTS, com suporte estendido até 2036. 

??? Linux Kernel wikipidia - Versões
    <iframe src="https://en.wikipedia.org/wiki/Linux_kernel" title="Linux Kernel" width="100%" height="300"> </iframe>

Pense que um desenvolvedor de um sistema embarcado, que vai criar toda uma infra dedicada não quer ficar ter que ajustando e validando tudo novamente só para ter a versão mais nova do kenrel. A ideia de usar uma com maior suporte é minimizar esforços.

O kernel utiliza o sistema de `tag` do git:

```bash
$ git tag
...
v2.6.11
v2.6.11-tree
v2.6.12
v2.6.12-rc2
v2.6.12-rc3
v2.6.12-rc4
v2.6.12-rc5
v2.6.12-rc6
v2.6.13-rc2
v2.6.14-rc2
v2.6.14-rc3
...
```

Note que revisões ímpares são para Karnel em estágio de desenvolvimento, e números pares para versão de produção, exemplo :

- **Linux 2.4.x** - Produção
- **Linux 2.5.x** - Desenvolvimento 
- **Linux 2.6.x** - Produção
- ....

Vamos criar um branch da versão v4.4, para  isso execute o comando a seguir:

``` bash
$ git checkout v4.4
$ git checkout -b 4.4-SoC
```

## Configurando o kernel 

Uma vez no branch `4.4-SoC` precisamos configurar o kernel para nosso processador (ARM) e fazer as configurações necessárias no kernel. Primeiramente iremos gerar um arquivo de configuração `.config` padrão para SoCs ARM Altera:

``` bash
$ export ARCH=arm        # indica a arquitetura do Kernel
$ make socfpga_defconfig # gera o arquivo padrão de configuração para SoC
```

!!! note
    As configuração do kernel ficam salvos no arquivo: `.config` na raiz do repositório. Quando executamos o comando `make socfpga_defconfig`, o mesmo é inicializado com algumas configurações padrões. Você pode dar uma olhada na pasta: [linux/arch/arm/configs/socfpga_defconfig](https://github.com/torvalds/linux/blob/master/arch/arm/configs/socfpga_defconfig).

Agora vamos configurar alguns parâmetros específicos do Kernel para a nossa aplicação:

``` bash
$ make ARCH=arm menuconfig 
```

!!! note
    Talvez seja necessário instalar o pacote **libncurses5-dev**

Esse comando irá abrir a interface de configuração do Kernel do Linux (existem outras opções: `make xconfig`; `make config`; `make gconfig`, ...). Essa interface permite selecionarmos várias configurações do Kernel. Agora iremos seguir o roteiro proposto no tutorial a seguir, traduzido de maneira reduzida nesse tutorial. 

- https://rocketboards.org/foswiki/Documentation/EmbeddedLinuxBeginnerSGuide

### Configurando 

1. Automatically append version information to the version string

- General Setup :arrow_right:
    - **Desabilite** : *Automatically append version information to the version string*
 
!!! note "Exclude/ Include" 
    Para **Desativar** utilize a letra `N` do teclado, para incluir a letra `Y`
 
!!! note "Dúvidas?" 
    A maioria dos parâmetros possui uma explicação, basta apertar `?` para ler a respeito.

!!! note 
    - ref :  https://rocketboards.org/foswiki/Documentation/EmbeddedLinuxBeginnerSGuide#8
    Go into the “General Setup” menu. Uncheck “Automatically append version information to the version string”. This will prevent the kernel from adding extra “version” information to the kernel. Whenever we try to dynamically load a driver (also called kernel modules, as discussed in a later section) the kernel will check to see if the driver was built with the same version of the source code as itself. If it isn’t, it will reject to load that driver. For development, it’s useful to disable these options to make it easier to test out different versions of drivers. In a production system however, it’s recommend to keep this option enabled and only use drivers that were compiled with the correct version of the kernel.
    I encourage you to peruse the options in the General Setup menu and see what’s available to you (hitting “?” to view the help info for the highlighted option). Of particular importance to us is the “Embedded System” option (turns on advanced features) and the type of SLAB allocator used (determines how memory will be dynamically allocated in the kernel). If you want to use an initial ram disk or ram filesystem that would be enabled here as well (these will be explained in the next section).
    (texto extraído da referência)

-------------------------------------

#### Enable loadable module support

- Volte para o menu principal (`<ESC>` `<ESC>`) :arrow_right:
    - Note que o *Enable loadable module support* está ativado.
    
!!! note ""
    Isso permite que o kernel seja modificado (pelo carregamento de drivers) após a sua execução. Isso será útil quando formos desenvolver nosso próprio device driver, sem a necessidade de recompilarmos o kernel toda vez que desejamos testar uma modificação no código. É essa configuração que permite utilizarmos USBs, SSDs, placas de rede via a possibilidade do carregamento de drivers de forma dinâmica pelo sistema operacional.

-------------------------------------

#### Support for large (2TB+) block devices and files

- No menu principal :arrow_right: Enable the block layer :arrow_right:
    - **Ative** : Support for large (2TB+) block devices and files

!!! note ""
    Essa opção irá permitir a utilização de partições do tipo EXT4. Se esquecer essa opção e o kernel tiver em uma partição EXT4 a mesma será montada como READ-ONLY.

-------------------------------------

#### The Extended 4 (ext4) filesystem 

- Menu principal :arrow_right: File systems :arrow_right: 
    - **Ative** : The Extended 4 (ext4) filesystem
    
!!! note ""
    Essa opção irá possibilitar que o kernel monte dispositivos formatados em EXT4. Pretendemos usar isso no SDCARD.

-------------------------------------

#### Altera SOCFPGA family

- Menu principal :arrow_right: System Type :arrow_right:
    - **Ative** : Altera SOCFGPA family
    
!!! note ""
    Isso indica para o kernel qual será o dispositivo que o mesmo será executado, note que esssa opção possui um novo menu onde podemos ativar ou não a suspensão para RAM.

-------------------------------------

#### Symmetric Multi-Processing
    
- Menu principal :arrow_right: Kernel Features :arrow_right:
    - **Ative**: Symmmetric Multi-Processing
    
!!! note ""
    Essa opção indica para o kernel que ele deve utilizar os dois cores presente no ARM HPS da FPGA. 

-------------------------------------

#### Device Drivers

- Menu principal :arrow_right: Device Drivers :arrow_right: 

!!! note ""
    Indica quais drivers serão compilados junto com o kernel, note que já temos configurado drivers de rede (Network device support); GPIO (GPIO Support); RTC; DMA; ... . Lembre que já inicializamos o `.config` com uma configuração padrão para SoCs Altera.

### Salvando

Aperte ESC duas vezes (`<ESC>`  `<ESC>`) e salve as configurações no arquivo `.config`

### `.config`

!!! note
    De uma olhada no arquivo `.config` gerado! As vezes é mais fácil editar direto nele, do que ter que abrir o menu de configuração e encontrar o local de ativar um módulo.
    

## Compilando

O makefile utiliza a variável `CROSS_COMPILE` para definir o toolchain que irá fazer a compilação do kernel, vamos definir como sendo o GCC do Linaro baixado recentemente:

```bash
$ export CROSS_COMPILE=$GCC_Linaro/arm-linux-gnueabihf-
```

Para compilarmos o kernel :

```bash
make ARCH=arm LOCALVERSION= zImage -j 4
```

!!! note
    -j4 executa a compilação em 4 threads, você pode ajustar esse valor para adequar ao seu processador.

!!! note "Dica"
    Adicione o `export CROSS_COMPILE=....` ao seu `.bashrc` para não ter que ficar digitando isso sempre que tiver que compilar o kernel.
    
!!! fail
    Caso aconteça algum erro de build, deve-se verificar o path do `CROSS_COMPILE` ou se existe alguma dependência que não foi satisfeita.

Esse comando faz com que o kernel do linux seja compilado em uma versão compactada que é auto-extraída. Outras opções seriam :

- Image : Binário do kernel
- zImage: versão compactada que possui *self-extracting*
- uImage: uma versão que já possui o bootloader uboot

1^: https://stackoverflow.com/questions/22322304/image-vs-zimage-vs-uimage 

o zImage é salvo em:

- `arch/arm/boot/zImage`

!!! note "zImage"
    Esse arquivo é o binário que contém o kernel do linux e será executado no sistema embarcado.
 
Agora devemos atualizar o kernel que está no SDCard, para isso segue o tutorial no `/info-SDcard.md/` : Atualizando o kernel

## Executando

Para verificar se tudo está certo, basta colocar o cartão de memória no kit e verificar a versão do kernel em execução:

```bash
$ uname -a
Linux buildroot 4.14.0 #1 SMP Mon Jul 16 21:22:58 -03 2018 armv7l GNU/Linux
```

### Mouse/ Teclado

!!! example 
    Plugue um mouse USB na placa e verifique se a mesma funciona! 

