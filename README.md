# Início rápido
## (http://wiki.debian.org/Simple-CDD/Howto)

Para experimentar o Simple-CDD, em um sistema Debian:
Instale o simple-cdd (como root):

 > # apt-get install simple-cdd

Crie um diretório de trabalho (como um usuário):
> $ mkdir ~ / my-simple-cdd
> $ cd ~ / my-simple-cdd'

## Construa um CD básico:

> $ build-simple-cdd

Isso criará um espelho de pacote parcial no diretório tmp / mirror,
e se tudo correr bem, uma imagem de CD .iso no diretório "imagens" quando
está terminado. Os registros são gerados em tmp / log e tmp / log / TOOL
documenta as variáveis ​​exportadas para cada ferramenta e o comando invocado
pelo módulo correspondente. Todas as variáveis ​​são documentadas em
simple_cdd / variables.py.

Por padrão, a versão de lançamento do CDD de destino é a mesma do host
versão. Você pode especificar o argumento opcional --dist para alterar o
versão de alvos. Por exemplo, pode ser etch, lenny, sid, etc.

Se esta etapa não funcionar, você precisa descobrir o porquê antes de tentar mais
coisas complicadas.

### Crie um perfil chamado NAME:
 > $ mkdir profiles
 > $ for p in list-of-packages-you-want-installed ; do echo $p >> profiles/NAME.packages ; done

Observe que você não deve incluir dependências de pacote, mas apenas os pacotes
você realmente quer.

Construa o CD com o perfil selecionado NAME:

 > $ build-simple-cdd --profiles NAME

Isso deve criar uma imagem de CD .iso no diretório "images" quando for
terminou com seu perfil personalizado.

Use qemu para testar:

> # apt-get install qemu
> $ build-simple-cdd --qemu --profiles NAME

Recursos opcionais:

Se você quiser pré-configuração debconf, coloque um arquivo compatível com debconf-set-selections
em profiles / NAME.preseed.

*Se você quiser um script de pós-instalação personalizado, coloque-o em profiles / NAME.postinst.

*Para mais opções:

> $ build-simple-cdd --help


# O menos rápido

Preparando um Diretório de Trabalho Simple-CDD

> mkdir my-cdd
> CD my-cdd


## profiles

crie o diretório de profiles:

>  mkdir profiles
>  
para fazer um perfil personalizado, pense em um nome para ele, como "x-basic".

edite ou crie arquivos no diretório "profiles", começando com o nome do perfil e terminando em .preseed, .packages, .downloads, etc. por exemplo:

'''
profiles / x-basic.preseed
profiles / x-basic.packages
profiles / x-basic.downloads
'''


 *.description

 breve descrição de uma linha do que o perfil faz

 * .packages
 
 pacotes instalados quando o perfil é selecionado. não inclua pacotes como
 como linux-image ou grub, pois o debian-installer tratará disso especialmente.

 *.downloads
 
 pacotes adicionais incluídos em um CD com o perfil, mas não instalados
 por simple-cdd (embora o debian-installer possa instalá-los)

 * .preseed
 
 questões debconf carregadas se o perfil for selecionado

 * .postinst
 
 script de pós-instalação específico do perfil. é executado na fase de "instalação final" de
 debian-installer.

 * .conf
 
 definições de configuração específicas do perfil. originado durante a construção do CD.


### para construir uma imagem de CD com o perfil x-basic:

> build-simple-cdd --profiles x-basic

ao instalar a partir do CD, um menu deve aparecer perguntando quais perfis você
deseja instalar. selecione todos os perfis que você deseja, e o debconf pré-configuração
os arquivos preencherão o banco de dados debconf e os pacotes serão instalados.


### Perfil Padrão

o perfil denominado "default" é especial, porque sempre é instalado.

modifique os arquivos profile / default. * com cuidado, pois o simple-cdd depende do
perfil padrão funcionando de certas maneiras (como instalar o
perfis de cdd simples .udeb).


# Pré-configuração Debconf

O preseed debconf é uma forma de pré-responder as questões feitas durante o pacote
instalação e debian-installer.

ele usa o formato debconf-set-selections. para mais informações sobre o formato:

 man debconf-set-selections

profiles / default.preseed é carregado após o CD debian-installer ser montado.
outros arquivos de pré-configuração de perfis são carregados quando o simples-cdd-profile .udeb é
instalado. algumas questões podem ter que ser passadas no prompt de boot (veja abaixo),
como eles são solicitados antes de qualquer um dos arquivos de pré-configuração serem carregados.

a seguinte questão é usada pelo simple-cdd, modifique por sua própria conta e risco:

 > d-i preseed / early_command


 # Seleção automática de perfis

para selecionar perfis automaticamente, em profiles / default.preseed, descomente o
linha:

 simple-cdd simple-cdd / profiles multiselect

e adicione todos os perfis que desejar, separados por vírgulas, ou seja:

 simple-cdd simple-cdd / profiles multiselect x-basic, ltsp

como alternativa, use a opção de linha de comando --auto-profiles:

 build-simple-cdd --profiles x-basic, ltsp --auto-profiles x-basic, ltsp


Seleção de idioma e país

para pré-selecionar o idioma e o país, é recomendado usar o --locale
opção de linha de comando:

 build-simple-cdd --locale pt_BR


Arquivos de configuração

para especificar um arquivo de configuração:

 > build-simple-cdd --conf my-cdd.conf

em my-cdd.conf, inclua valores como


'''
 locale = pt_BR
 perfis = "x-básico, ltsp"
 auto_profiles = "x-basic, ltsp"
 debian_mirror = "http: //my.local.mirror/debian"
 '''


você também pode especificar arquivos de configuração por perfil, em

profiles / <perfil> .conf.


Passando nos parâmetros do prompt de inicialização

para passar os parâmetros de inicialização, defina KERNEL_PARAMS em um arquivo de configuração. a
o exemplo a seguir adiciona o arquivo default.preseed (necessário para simple-cdd para
função) e desativa o gerenciamento de energia (acpi):

 export KERNEL_PARAMS = "preseed / file = / cdrom / simple-cdd / default.preseed acpi = off"


Construa o CD

 build-simple-cdd

agora as ferramentas de espelhamento irão baixar muitos e muitos arquivos.

então a imagem do CD será construída e aparecerá como um arquivo no diretório "imagens",
como "debian-40-i386-CD-1.iso"


Testando com Qemu

você pode testar se sua imagem funciona usando qemu ...

 apt-get install qemu

 build-simple-cdd --qemu --profiles x-basic

isso irá construir a imagem do CD usando o perfil x-basic, execute o qemu para instalá-lo,
e execute o qemu para a inicialização do sistema.


Testando com Qemu no modo não gráfico

testei este código inteiramente com qemu em uma conexão SSH lenta,
então eu precisava descobrir como fazer o console serial funcionar ...

 build-simple-cdd --qemu --profiles x-basic --serial-console


Mais diversão e aventuras

Ganchos de pós e pré-instalação

se você precisa fazer alguma personalização que não pode ser tratada pelo debconf
preseed, escreva um script fazendo o que você precisa fazer ... e copie esse script para
qualquer:

 default.postinst (para todas as instalações)

ou

 <perfil> .postinst (para pós-instalação específica do perfil)


Particionamento Totalmente Automático

já que sobrescrever automaticamente os dados no disco rígido pode ser
destrutivo, ele é desabilitado por padrão.

para habilitá-lo, edite profiles / default.preseed e descomente os três seguintes
linhas:

 d-i partman / confirm_write_new_label boolean true
 d-i partman / choose_partition select Concluir o particionamento e gravar as alterações no disco
 d-i partman / confirm boolean true

Usando contrib e non-free

se você deseja adicionar pacotes do contrib ou non-free, adicione-os ao
mirror_components em um arquivo de configuração:

 mirror_components = "contrib principal não livre"

se você estiver usando um espelho que usa componentes diferentes, adicione-os
mirror_components_extra em um arquivo de configuração:

 debian_mirror_extra = "http://some.mirror.org/debian/"
 mirror_components_extra = "contrib non-free"
