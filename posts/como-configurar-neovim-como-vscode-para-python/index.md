# Como Configurar Neovim Como VsCode


## Introducción

&lt;a href=&#34;https://neovim.io/&#34; target=&#34;_blank&#34;&gt;Neovim&lt;/a&gt; es VIM pero mejorado. La documentación oficial lo define como:

&gt; Neovim es un editor de texto basado en Vim diseñado para la extensibilidad y usabilidad, para fomentar nuevas aplicaciones y contribuciones.

## Instalación

### Prerequisitos

Como lo dice el título de este artículo, me estaré concentrando en configurar Neovim como IDE para python, entonces voy a asumir que ya tienes Python y Pip instalados en tu sistema.

#### Python, Pip y Pipenv

Para verificar que tengas Python instalado corre uno de los siguientes comandos en la consola:

```bash
$ python --version
$ python3 --version
```

El comando anterior deberá decirte la versión de Python instalada en tu sistema

Para verificar si Pip está instalado, solo ejecuta pip en la consola. El comando deberá regresar la &#34;ayuda&#34;.

```bash
$ pip

Usage:
  pip &lt;command&gt; [options]

Commands:
  install                     Install packages.
  download                    Download packages.
  uninstall                   Uninstall packages.
  freeze                      Output installed packages in requirements format.
  list                        List installed packages.
  show                        Show information about installed packages.
  check                       Verify installed packages have compatible dependencies.
  config                      Manage local and global configuration.
  search                      Search PyPI for packages.
  cache                       Inspect and manage pip&#39;s wheel cache.
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.
  completion                  A helper command used for command completion.
  debug                       Show information useful for debugging.
  help                        Show help for commands.
```

Si no tienes instalado pip, lo puedes instalar ejectando:

```bash
$ sudo apt-get install python3-pip
```

Para verificar si Pipenv está instalado, puedes ejecutar:

```bash
$ pipenv --version

pipenv, version 2020.11.15
```

Para instalar Pipenv, utiliza el siguiente comando:

```bash
$ pip3 install pipenv
```

#### Git

También vas a necesitar git para poder instalar el manejador de plugins &#34;https://github.com/junegunn/vim-plug&#34;

Si no lo tienes instalado, lo puedes instalar utilizando:

```bash
$ sudo apt-get install git
```

#### Neovim

Por supuesto, debemos tener instalado Neovim con soporte para python, si no lo tienes aún, lo puedes instalar de la siguiente manera:

```bash
$ sudo apt-get install neovim python3-neovim
```

#### Node.js

Para poder instalar los plugins que nos van a permitir utilizar Neovim como IDE y reemplazar de una vez por todas VsCode, es necesario también instalar Node.js.

```bash
$ sudo apt-get install npm
```

#### Nerdfont (Tipo de Letra)

Para poder visualizar los iconos en la terminal es necesario tener un tipo de letra que lo soporte, en lo personal a mi me gustan las Nerdfonts.

Puedes descargar &lt;a href=&#34;https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/DejaVuSansMono.zip&#34;&gt;DejaVuSansMono Nerd Font&lt;/a&gt;.

Una vez que hayas descargado el archivo, descomprimelo y copia los archivos (\*.ttf) al siguiente directorio:

```
$HOME/.local/share/fonts/
```

Si por alguna razón no tienes el directorio, lo puedes crear.

Ya que hayas copiado los archivos, para poder utilizar este nuevo tipo de letra, debes ejecutar:

```bash
$ fc-cache -fv
```

**Asegurate de configurar este nuevo tipo de letra en la terminal que estés utilizando:**

![Terminal](/images/posts/neovim-python/nerdfont-terminal.png)

## Configuración

### Directorio nvim

Aunque puedes tener toda la configuración en tu archivo **init.vim** yo prefiero tener archivos separados para una mejor organización y fácil manejo.

Crea la siguiente estructura de direcorios:

```
📂 $HOME/.config/nvim/
├──📁 general
├──📁 keymapping
├──📁 nvim-plugins
└──📁 plug-config
```

```bash
mkdir $HOME/.config/nvim
mkdir $HOME/.config/nvim/general
mkdir $HOME/.config/nvim/keymapping
mkdir $HOME/.config/nvim/nvim-plugins
mkdir $HOME/.config/nvim/plug-config
```

### Ambiente de Python para Neovim

Vamos a crear un ambiente virtual dentro de la carpeta de configuración de Neovim. Para eso, en la terminal tecleamos lo siguiente:

```bash
cd ~/.config/nvim

pipenv install
```

Copia la ruta donde se creó el ambiente virtual pues lo usaremos en el siguiente paso:

![pipenv output](/images/posts/neovim-python/pipenv-install.png)

### Archivos .vim

#### general/settings.vim

```settings.vim
syntax enable

set guicursor=                                     &#34; Disable blinking for the n-v-c modes
set termguicolors
set guioptions-=T                                   &#34; No Tool bar

set cursorline                                     &#34; Highlight the current line

set hidden                                         &#34; When on a buffer becomes hidden when it is abandoned
set path&#43;=**
set nowrap
set encoding=UTF-8

set number relativenumber

set smartindent
set smarttab
set tabstop=4 softtabstop=4
set shiftwidth=4
set expandtab
set smartcase
set incsearch
set nohlsearch
set completeopt=menuone,noinsert,noselect
set signcolumn=yes
set colorcolumn=80
highlight ColorColumn ctermbg=0 guibg=lightgrey

set noswapfile
set nobackup
set undofile
execute &#39;set undodir=&#39; . g:nvim_data_root . &#39;/undodir&#39;

&#34; Python virtual environment detection in linux
let g:loaded_python_provider = 0
&#34; Ruta donde se creó el ambiente de Python con pipenv.
let g:python3_host_prog = &#39;/home/{usuario}/.local/share/virtualenvs/{nvim-999999}/bin/python&#39;

```

### Plugins

#### Instala vim-plug

Para instalar vim-plug vamos a ejecutar el siguiente comando:

```bash
sh -c &#39;curl -fLo &#34;${XDG_DATA_HOME:-$HOME/.local/share}&#34;/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim&#39;
```

#### $HOME/.config/nvim/nvim-plugins/plugins.vim

```plugins.vim
call plug#begin(stdpath(&#39;data&#39;).&#39;/plugged&#39;)

    &#34; NERDTree
    Plug &#39;preservim/nerdtree&#39;

    &#34; VIM-DEVICONS icons for NERDTree
    Plug &#39;ryanoasis/vim-devicons&#39;

    &#34; Conquer Of Completion
    Plug &#39;neoclide/coc.nvim&#39;, {&#39;branch&#39;: &#39;release&#39;}

    &#34; Surround.vim
    Plug &#39;tpope/vim-surround&#39;

    &#34; Airline status bar
    Plug &#39;vim-airline/vim-airline&#39;
    Plug &#39;vim-airline/vim-airline-themes&#39;

    &#34; NERDCommenter
    Plug &#39;preservim/nerdcommenter&#39;

call plug#end()

```

#### $HOME/.config/nvim/plug-config/coc.vim

El siguiente es un ejemplo funcional tomado de &lt;a href=&#34;https://github.com/neoclide/coc.nvim&#34; target=&#34;_blank&#34;&gt;la documentación oficial&lt;/a&gt; en github.

```coc.vim
let g:coc_global_extensions = [
    \ &#39;coc-snippets&#39;,
    \ &#39;coc-pairs&#39;,
    \ &#39;coc-python&#39;,
    \ &#39;coc-prettier&#39;,
    \ &#39;coc-vimlsp&#39;,
    \ &#39;coc-marketplace&#39;,
\ ]

&#34; TextEdit might fail if hidden is not set.
set hidden

&#34; Some servers have issues with backup files, see #649.
set nobackup
set nowritebackup

&#34; Give more space for displaying messages.
set cmdheight=2

&#34; Having longer updatetime (default is 4000 ms = 4 s) leads to noticeable
&#34; delays and poor user experience.
set updatetime=300

&#34; Don&#39;t pass messages to |ins-completion-menu|.
set shortmess&#43;=c

&#34; Always show the signcolumn, otherwise it would shift the text each time
&#34; diagnostics appear/become resolved.
if has(&#34;patch-8.1.1564&#34;)
  &#34; Recently vim can merge signcolumn and number column into one
  set signcolumn=number
else
  set signcolumn=yes
endif

&#34; Use tab for trigger completion with characters ahead and navigate.
&#34; NOTE: Use command &#39;:verbose imap &lt;tab&gt;&#39; to make sure tab is not mapped by
&#34; other plugin before putting this into your config.
inoremap &lt;silent&gt;&lt;expr&gt; &lt;TAB&gt;
      \ pumvisible() ? &#34;\&lt;C-n&gt;&#34; :
      \ &lt;SID&gt;check_back_space() ? &#34;\&lt;TAB&gt;&#34; :
      \ coc#refresh()
inoremap &lt;expr&gt;&lt;S-TAB&gt; pumvisible() ? &#34;\&lt;C-p&gt;&#34; : &#34;\&lt;C-h&gt;&#34;

function! s:check_back_space() abort
  let col = col(&#39;.&#39;) - 1
  return !col || getline(&#39;.&#39;)[col - 1]  =~# &#39;\s&#39;
endfunction

&#34; Use &lt;c-space&gt; to trigger completion.
if has(&#39;nvim&#39;)
  inoremap &lt;silent&gt;&lt;expr&gt; &lt;c-space&gt; coc#refresh()
else
  inoremap &lt;silent&gt;&lt;expr&gt; &lt;c-@&gt; coc#refresh()
endif

&#34; Make &lt;CR&gt; auto-select the first completion item and notify coc.nvim to
&#34; format on enter, &lt;cr&gt; could be remapped by other vim plugin
inoremap &lt;silent&gt;&lt;expr&gt; &lt;cr&gt; pumvisible() ? coc#_select_confirm()
                              \: &#34;\&lt;C-g&gt;u\&lt;CR&gt;\&lt;c-r&gt;=coc#on_enter()\&lt;CR&gt;&#34;

&#34; Use `[g` and `]g` to navigate diagnostics
&#34; Use `:CocDiagnostics` to get all diagnostics of current buffer in location list.
nmap &lt;silent&gt; [g &lt;Plug&gt;(coc-diagnostic-prev)
nmap &lt;silent&gt; ]g &lt;Plug&gt;(coc-diagnostic-next)

&#34; GoTo code navigation.
nmap &lt;silent&gt; gd &lt;Plug&gt;(coc-definition)
nmap &lt;silent&gt; gy &lt;Plug&gt;(coc-type-definition)
nmap &lt;silent&gt; gi &lt;Plug&gt;(coc-implementation)
nmap &lt;silent&gt; gr &lt;Plug&gt;(coc-references)

&#34; Use K to show documentation in preview window.
nnoremap &lt;silent&gt; K :call &lt;SID&gt;show_documentation()&lt;CR&gt;

function! s:show_documentation()
  if (index([&#39;vim&#39;,&#39;help&#39;], &amp;filetype) &gt;= 0)
    execute &#39;h &#39;.expand(&#39;&lt;cword&gt;&#39;)
  elseif (coc#rpc#ready())
    call CocActionAsync(&#39;doHover&#39;)
  else
    execute &#39;!&#39; . &amp;keywordprg . &#34; &#34; . expand(&#39;&lt;cword&gt;&#39;)
  endif
endfunction

&#34; Highlight the symbol and its references when holding the cursor.
autocmd CursorHold * silent call CocActionAsync(&#39;highlight&#39;)

&#34; Symbol renaming.
nmap &lt;leader&gt;rn &lt;Plug&gt;(coc-rename)

&#34; Formatting selected code.
xmap &lt;leader&gt;f  &lt;Plug&gt;(coc-format-selected)
nmap &lt;leader&gt;f  &lt;Plug&gt;(coc-format-selected)

augroup mygroup
  autocmd!
  &#34; Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction(&#39;formatSelected&#39;)
  &#34; Update signature help on jump placeholder.
  autocmd User CocJumpPlaceholder call CocActionAsync(&#39;showSignatureHelp&#39;)
augroup end

&#34; Applying codeAction to the selected region.
&#34; Example: `&lt;leader&gt;aap` for current paragraph
xmap &lt;leader&gt;a  &lt;Plug&gt;(coc-codeaction-selected)
nmap &lt;leader&gt;a  &lt;Plug&gt;(coc-codeaction-selected)

&#34; Remap keys for applying codeAction to the current buffer.
nmap &lt;leader&gt;ac  &lt;Plug&gt;(coc-codeaction)
&#34; Apply AutoFix to problem on the current line.
nmap &lt;leader&gt;qf  &lt;Plug&gt;(coc-fix-current)

&#34; Map function and class text objects
&#34; NOTE: Requires &#39;textDocument.documentSymbol&#39; support from the language server.
xmap if &lt;Plug&gt;(coc-funcobj-i)
omap if &lt;Plug&gt;(coc-funcobj-i)
xmap af &lt;Plug&gt;(coc-funcobj-a)
omap af &lt;Plug&gt;(coc-funcobj-a)
xmap ic &lt;Plug&gt;(coc-classobj-i)
omap ic &lt;Plug&gt;(coc-classobj-i)
xmap ac &lt;Plug&gt;(coc-classobj-a)
omap ac &lt;Plug&gt;(coc-classobj-a)

&#34; Remap &lt;C-f&gt; and &lt;C-b&gt; for scroll float windows/popups.
if has(&#39;nvim-0.4.0&#39;) || has(&#39;patch-8.2.0750&#39;)
  nnoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-f&gt; coc#float#has_scroll() ? coc#float#scroll(1) : &#34;\&lt;C-f&gt;&#34;
  nnoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-b&gt; coc#float#has_scroll() ? coc#float#scroll(0) : &#34;\&lt;C-b&gt;&#34;
  inoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-f&gt; coc#float#has_scroll() ? &#34;\&lt;c-r&gt;=coc#float#scroll(1)\&lt;cr&gt;&#34; : &#34;\&lt;Right&gt;&#34;
  inoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-b&gt; coc#float#has_scroll() ? &#34;\&lt;c-r&gt;=coc#float#scroll(0)\&lt;cr&gt;&#34; : &#34;\&lt;Left&gt;&#34;
  vnoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-f&gt; coc#float#has_scroll() ? coc#float#scroll(1) : &#34;\&lt;C-f&gt;&#34;
  vnoremap &lt;silent&gt;&lt;nowait&gt;&lt;expr&gt; &lt;C-b&gt; coc#float#has_scroll() ? coc#float#scroll(0) : &#34;\&lt;C-b&gt;&#34;
endif

&#34; Use CTRL-S for selections ranges.
&#34; Requires &#39;textDocument/selectionRange&#39; support of language server.
nmap &lt;silent&gt; &lt;C-s&gt; &lt;Plug&gt;(coc-range-select)
xmap &lt;silent&gt; &lt;C-s&gt; &lt;Plug&gt;(coc-range-select)

&#34; Add `:Format` command to format current buffer.
command! -nargs=0 Format :call CocAction(&#39;format&#39;)

&#34; Add `:Fold` command to fold current buffer.
command! -nargs=? Fold :call     CocAction(&#39;fold&#39;, &lt;f-args&gt;)

&#34; Add `:OR` command for organize imports of the current buffer.
command! -nargs=0 OR   :call     CocAction(&#39;runCommand&#39;, &#39;editor.action.organizeImport&#39;)

&#34; Add (Neo)Vim&#39;s native statusline support.
&#34; NOTE: Please see `:h coc-status` for integrations with external plugins that
&#34; provide custom statusline: lightline.vim, vim-airline.
set statusline^=%{coc#status()}%{get(b:,&#39;coc_current_function&#39;,&#39;&#39;)}

&#34; Mappings for CoCList
&#34; Show all diagnostics.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;a  :&lt;C-u&gt;CocList diagnostics&lt;cr&gt;
&#34; Manage extensions.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;e  :&lt;C-u&gt;CocList extensions&lt;cr&gt;
&#34; Show commands.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;c  :&lt;C-u&gt;CocList commands&lt;cr&gt;
&#34; Find symbol of current document.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;o  :&lt;C-u&gt;CocList outline&lt;cr&gt;
&#34; Search workspace symbols.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;s  :&lt;C-u&gt;CocList -I symbols&lt;cr&gt;
&#34; Do default action for next item.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;j  :&lt;C-u&gt;CocNext&lt;CR&gt;
&#34; Do default action for previous item.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;k  :&lt;C-u&gt;CocPrev&lt;CR&gt;
&#34; Resume latest coc list.
nnoremap &lt;silent&gt;&lt;nowait&gt; &lt;space&gt;p  :&lt;C-u&gt;CocListResume&lt;CR&gt;
```

#### $HOME/.config/nvim/plug-config/nerdcommenter.vim

```nerdcommenter.vim
&#34; Add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1

&#34; Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

&#34; Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = &#39;left&#39;

&#34; Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

&#34; Add your own custom formats or override the defaults
let g:NERDCustomDelimiters = { &#39;c&#39;: { &#39;left&#39;: &#39;/**&#39;,&#39;right&#39;: &#39;*/&#39;  }  }

&#34; Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

&#34; Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

&#34; Enable NERDCommenterToggle to check all selected lines is commented or not
 let g:NERDToggleCheckAllLines = 1&#34;

```

#### $HOME/.config/nvim/plug-config/airline.vim

```airline.vim
&#34; enable tabline
let g:airline#extensions#tabline#enabled = 1
&#34; let g:airline#extensions#tabline#left_sep = &#39;&#39;
&#34; let g:airline#extensions#tabline#left_alt_sep = &#39;&#39;
&#34; let g:airline#extensions#tabline#right_sep = &#39;&#39;
&#34; let g:airline#extensions#tabline#right_alt_sep = &#39;&#39;

&#34; enable powerline fonts
let g:airline_powerline_fonts = 1
&#34; let g:airline_left_sep = &#39;&#39;
&#34; let g:airline_right_sep = &#39;&#39;

&#34; Always show tabs
set showtabline=2

&#34; We don&#39;t need to see things like -- INSERT -- anymore
set noshowmode
```

#### $HOME/.config/nvim/keymapping/keys.vim

```keys.vim
&#34; Edit vimrc faster
nnoremap &lt;Leader&gt;v :e $MYVIMRC&lt;cr&gt;

&#34; Reload init.vim
nnoremap &lt;silent&gt; &lt;Leader&gt;&lt;Leader&gt; :source $MYVIMRC&lt;cr&gt;

&#34;Change working directory
nnoremap &lt;leader&gt;cd :cd %:p:h&lt;cr&gt;

&#34;Mapping NERDTree
map &lt;C-n&gt; :NERDTreeToggle&lt;cr&gt;

&#34; Use alt &#43; hjkl to resize windows
nnoremap &lt;M-j&gt;  :resize -2&lt;CR&gt;
nnoremap &lt;M-k&gt;  :resize &#43;2&lt;CR&gt;
nnoremap &lt;M-h&gt;  :vertical resize -2&lt;CR&gt;
nnoremap &lt;M-l&gt;  :vertical resize &#43;2&lt;CR&gt;


&#34; NerdCommenter Toggle remap to CTRL&#43;/
vmap &lt;C-_&gt; &lt;Plug&gt;NERDCommenterToggle&lt;cr&gt;
nmap &lt;C-_&gt; &lt;Plug&gt;NERDCommenterToggle&lt;cr&gt;

&#34; Start Terminal using PowerShell 7 (Preview)
&#34; For default shell, remove &#34;://pwsh.exe&#34;
&#34; You can customize the shell by replacing &#34;pwsh.exe&#34; with your shell for example:
&#34;       :edit term://bash
&#34;       :vsplit term://top
&#34; For more :help terminal
nmap &lt;leader&gt;t :vsplit term://bash&lt;cr&gt;

&#34; Move between buffers
nnoremap &lt;leader&gt;&lt;tab&gt; :bn&lt;cr&gt;
nnoremap &lt;leader&gt;&lt;S-tab&gt; :bp&lt;cr&gt;

&#34; Closing and hiding current buffer
nnoremap &lt;leader&gt;w :bd&lt;cr&gt;
```

#### $HOME/.config/nvim/init.vim

```init.vim
let g:nvim_data_root = stdpath(&#39;data&#39;)
let g:nvim_config_root = stdpath(&#39;config&#39;)
let g:config_file_list = [&#39;/nvim-plugins/plugins.vim&#39;,
    \ &#39;/general/settings.vim&#39;,
    \ &#39;/keymapping/keys.vim&#39;,
    \ &#39;/plug-config/airline.vim&#39;,
    \ &#39;/plug-config/coc.vim&#39;,
    \ &#39;/plug-config/nerdcommenter.vim&#39;,
    \ ]

for f in g:config_file_list
    execute &#39;source &#39; . g:nvim_config_root . f
endfor
```

## Reiniciar Neovim e instalar los plugins

Una vez que guardes los cambios, debes salir de Neovim y volver a entrar, luego ejecutar **:PlugInstall**

Esto instalará los plugins y las extensiones de CConquer of Completion (coc).

## Paso Final: Configura coc-python

Para que todo funcione correctamente debes instalar de manera global una libreria de python llamada &lt;a href=&#34;https://pypi.org/project/jedi/&#34; target=&#34;_blank&#34;&gt;&#34;jedi&#34;&lt;/a&gt;. Al instalarlo de manera global te evitarás tener que instalarlo en cada uno de tus ambientes virtuales.

Para Instalar:

```bash
$ pip install jedi
```

Una vez instalada, hay que configurar CoC. Primero copia la ruta donde se instaló jedi, para saber esa ruta ejecuta lo siguiente en la terminal:

```bash
$ pip show jedi

Name: jedi
Version: 0.17.2
Summary: An autocompletion tool for Python that can be used for text editors.
Home-page: https://github.com/davidhalter/jedi
Author: David Halter
Author-email: davidhalter88@gmail.com
License: MIT
Location: /usr/lib/python3.9/site-packages &lt;== Esta es la ruta que debes copiar en el siguiente archivo de configuración
Requires: parso
Required-by:
```

Abre Neovim y ejecuta **:CocConfig**

Esto abrirá un buffer vacío, este buffer es donde &lt;a href=&#34;https://github.com/neoclide/coc.nvim/wiki/Using-the-configuration-file&#34; target=&#34;_blank&#34;&gt;configuras CoC&lt;/a&gt;

Agrega lo siguiente, reemplazando la ruta por la ruta donde está instalado &#34;jedi&#34; en tu sistema:

```coc-settings.json
{
    &#34;python.jediPath&#34;: &#34;/usr/lib/python3.9/site-packages&#34;
}
```

Guarda el archivo, cierra Neovim y vuelve a abrir.

Listo, hasta aquí tienes una configuración lista para trabajar con Neovim y reemplazar VSCode.

## Bonus

### Gruvbox 💻

En lo personal, el mejor tema para programar en Neovim, es &lt;a href=&#34;https://github.com/morhetz/gruvbox&#34; target=&#34;_blank&#34;&gt; Gruvbox&lt;/a&gt;, la combinación de colores se me hace seria y sobretodo muy cómodo para la vista, simple y sencillamente... ¡Me Encanta!

Primero agregamos el plugin en $HOME/.config/nvim/nvim-plugins/plugins.vim

```plugins.vim
    &#34; Other themes
    Plug &#39;morhetz/gruvbox&#39;
```

Recuerda que una vez que guardes los cambios, debes salir de Neovim y volver a entrar, luego correr :PlugInstall

Después hay que editar $HOME/.config/nvim/general/settings.vim para agregar gruvbox como tema por defecto.

```settings.vim
&#34; Gruvbox por defecto
colorscheme gruvbox
let g:gruvbox_contrast_dark=&#39;hard&#39;
set background=dark
```

Por último, vamos a configurar Airline con el mismo tema de colores. Para eso hay que editar $HOME/.config/nvim/plug-config/airline.vim

```airline.vim
&#34;Gruvbox
let g:airline_theme=&#39;gruvbox&#39;
```

## Conclusión

![Neovim Como VSCode](/images/posts/neovim-python/neovim-como-vscode.png)

&lt;a href=&#34;https://neovim.io&#34; target=&#34;_blank&#34;&gt;Neovim&lt;/a&gt; es un editor de textos ligero y sobretodo muy &#34;configurable&#34;, al principio puede parecer intimidante pero con el tiempo y la práctica se vuelve una herramienta muy útil para el desarrollo.

Si deseas conocer que otras extensiones existen para CoC, visita la &lt;a href=&#34;https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions&#34; target=&#34;_blank&#34;&gt;wiki oficial&lt;/a&gt; donde encontraras basta información.

Si tienes algún comentario o recomendación de algún artículo que te gustaría que cubriera en este blog, no dudes en mandarme un mensaje en twitter a &lt;a href=&#34;https://twitter.com/jorgers&#34; target=&#34;\_blank&#34;&gt;@JorgeRS&lt;/a&gt;

Si te gustó este artículo por favor compartelo en cualquiera de las redes sociales utilizando los botones más abajo.


---

> Autor:   
> URL: http://localhost:1313/posts/como-configurar-neovim-como-vscode-para-python/  

