" =============================================================================
" 🔢 UI & Line Numbers
" =============================================================================
set number              " Show absolute line number for the current line
set relativenumber      " Show relative numbers for easier jumping

" =============================================================================
" ⌨️ Indentation & Tabs
" =============================================================================
set expandtab           " Convert tabs to spaces
set smarttab            " Insert blanks according to shiftwidth
set tabstop=4           " Number of spaces a tab counts for
set shiftwidth=4        " Number of spaces for auto-indenting
set softtabstop=4       " Number of spaces a tab counts for while editing
set autoindent          " Copy indent from current line when starting a new one

" Language specific overrides
autocmd FileType yaml setlocal ai ts=2 sw=2 et

" =============================================================================
" 🔍 Search & Performance
" =============================================================================
set smartcase           " Override 'ignorecase' if search contains capitals
set noswapfile          " Disable swap files for better performance
set nobackup            " Disable backup files
set nowritebackup       " Disable write backups

" =============================================================================
" 🎨 Appearance & Syntax
" =============================================================================
syntax on               " Enable syntax highlighting
colorscheme desert      " Set the color scheme

" =============================================================================
" 🚀 Custom Key Mappings
" =============================================================================

" Tab indentation in Normal and Visual mode
nnoremap <Tab> >>
nnoremap <S-Tab> <<
xnoremap <Tab> >
xnoremap <S-Tab> <

" 🎯 Auto-centering: Keep the cursor in the middle of the screen
" Vertical movement
nnoremap j jzz
nnoremap k kzz
nnoremap gj gjzz
nnoremap gk gkzz

" Paragraph & Bracket jumping
nnoremap } }zz
nnoremap { {zz
nnoremap ) )zz
nnoremap ( (zz
nnoremap ]] ]]zz
nnoremap [[ [[zz
nnoremap ][ ][zz
nnoremap [] []zz

" Top & Bottom jumping
nnoremap G Gzz
nnoremap gg ggzz

" Page scrolling
nnoremap <C-d> <C-d>zz
nnoremap <C-u> <C-u>zz
nnoremap <C-f> <C-f>zz
nnoremap <C-b> <C-b>zz
nnoremap <C-e> <C-e>zz
nnoremap <C-y> <C-y>zz

" Search result centering (zz) and opening folds (zv)
nnoremap n nzzzv
nnoremap N Nzzzv
nnoremap * *zzzv
nnoremap # #zzzv
nnoremap g* g*zzzv
nnoremap g# g#zzzv

" High/Middle/Low screen jumps
nnoremap H Hzz
nnoremap M Mzz
nnoremap L Lzz

" Visual mode centering
xnoremap j jzz
xnoremap k kzz
xnoremap } }zz
xnoremap { {zz
xnoremap G Gzz
xnoremap gg ggzz
