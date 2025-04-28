Basic
- [ ] to-do
- [/] incomplete
- [x] done
- [-] canceled
- [>] forwarded
- [<] scheduling
 Extras
- [?] question
- [!] important
- [*] star
- ["] quote
- [l] location
- [b] bookmark
- [i] information
- [S] savings
- [I] idea
- [p] pros
- [c] cons
- [f] fire
- [k] key
- [w] win
- [u] up
- [d] down
- [D] draft pull request
- [P] open pull request
- [M] merged pull request


# 1.  Disable ipv6
sudo vi /etc/default/grub
	GRUB_CMDLINE_LINUX="ipv6.disable=1"
sudo update-grub

# 2. Setup network for Beagble Bone and start to transfer data
sudo find . -name "ip_forward"

Share internet from VM to Beagble Bone Black:
A. Setup network for Beagble Bone
Step 1: BBB

sudo vi /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
sudo route add default gw 192.168.7.1 usb0
sudo route add default gw 192.168.6.1 usb1


Step 2: Host PC

echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 --jump MASQUERADE

sudo iptables --append FORWARD --in-interface enxf4b8987a7af7 --jump ACCEPT
B. start to transfer data
Step 3:
host pc:
sudo exportfs -rav
or
sudo exportfs -rv

BBB:
sudo mount 192.168.1.61:/home/truonglv/BBB/code /home/debian/drivers
sudo umount /home/debian/out

;sudo mount 192.168.1.61:/srv/nfs/bin_code /home/debian/bin_files

# 3. Command in NeoVim

## 3.1. Toggle NERDTree với Ctrl+n
```
nnoremap <C-n> :NERDTreeToggle<CR>
```

Phím tắt: Ctrl + n
Chức năng: Bật hoặc tắt NERDTree, một plugin hiển thị cây thư mục.
## 3.2. Sử dụng Tab để trigger gợi ý

```
inoremap <silent><expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <silent><expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"
```
Phím tắt: Tab và Shift + Tab
Chức năng: Khi popup menu (danh sách gợi ý) hiển thị, Tab sẽ di chuyển đến gợi ý tiếp theo (\<C-n>) và Shift + Tab sẽ di chuyển đến gợi ý trước đó (\<C-p>). Nếu popup menu không hiển thị, Tab và Shift + Tab sẽ hoạt động như bình thường.
## 3.2. Sử dụng Enter để chọn gợi ý
```
inoremap <silent><expr> <CR> pumvisible() ? coc#_select_confirm() : "\<CR>"
```
Phím tắt: Enter
Chức năng: Khi popup menu hiển thị, Enter sẽ chọn gợi ý hiện tại (coc#_select_confirm()). Nếu popup menu không hiển thị, Enter sẽ hoạt động như bình thường (chèn dòng mới).
## 3.4. Gọi lại gợi ý khi cần
```
inoremap <silent><expr> <C-Space> coc#refresh()
```
Phím tắt: Ctrl + Space
Chức năng: Gọi lại danh sách gợi ý thủ công.
# 4 PlugInstall ( Neo Vim)
## 4..1. Install vim-plug

https://nvchad.com/docs/quickstart/install

If you haven’t installed vim-plug yet, run the following command:
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

## 4.2 Neovim 
Gỡ cài đặt phiên bản cũ của Neovim (nếu có):
```
sudo apt remove neovim -y
```
Tải xuống tệp tarball của Neovim 0.8.0 từ trang GitHub:
```
 wget https://github.com/neovim/neovim/releases/download/v0.10.0/nvim-linux64.tar.gz
```

Giải nén tệp tarball và di chuyển nó đến thư mục /usr/local:
```
tar xzvf nvim-linux64.tar.gz
sudo mv nvim-linux64 /usr/local/nvim
```

Tạo liên kết tượng trưng (symbolic link) để dễ dàng truy cập Neovim:
```
sudo ln -s /usr/local/nvim/bin/nvim /usr/bin/nvim
```

Kiểm tra phiên bản Neovim để đảm bảo cài đặt thành công:
```
nvim --version
```

## 4.3 Node.js 
Để cài đặt Node.js trên Ubuntu 20.04.6, bạn có thể làm theo các bước sau:

Cách 1: Sử dụng APT từ kho phần mềm mặc định của Ubuntu
Cập nhật danh sách gói:
```
sudo apt update
```

Cài đặt Node.js:
```
sudo apt install nodejs
```

Kiểm tra phiên bản Node.js:
```
node -v
```

Cài đặt npm (Node Package Manager):
```
sudo apt install npm
```

# 5. init.vim
## 5.0 Download plugins

```
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## 5.1 link git
https://github.com/Truong0905/nvim


```
call plug#begin('~/.local/share/nvim/plugged')

" Add plugins here
Plug 'preservim/nerdtree'  " File tree explorer
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }  " Fuzzy finder
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'neovim/nvim-lspconfig'  " LSP configuration
Plug 'tpope/vim-surround'  " Surround text objects
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}  " Treesitter for better syntax highlighting
Plug 'nvim-telescope/telescope.nvim', {'tag': '0.1.0'}  " Fuzzy finder
Plug 'nvim-lua/plenary.nvim'  " Dependency for telescope
Plug 'lewis6991/gitsigns.nvim'  " Git integration
Plug 'hoob3rt/lualine.nvim'  " Status line

call plug#end()

" Enable line numbers
set number

" Enable syntax highlighting
syntax on

" Set tab width
set tabstop=4
set shiftwidth=4
set expandtab

" Enable line wrapping
set wrap

" Toggle NERDTree with Ctrl+n
nnoremap <C-n> :NERDTreeToggle<CR>

" Use tab to trigger suggestions
inoremap <silent><expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <silent><expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"

" Use Enter to confirm suggestions
inoremap <silent><expr> <CR> pumvisible() ? coc#_select_confirm() : "\<CR>"

" Manually trigger suggestions
inoremap <silent><expr> <C-Space> coc#refresh()
```

## 5.2 command 
```
:source $MYVIMRC
:PlugInstall
```
# 6. Lỗi bạn gặp phải khi chạy lệnh make menuconfig 
Lỗi bạn gặp phải khi chạy lệnh make menuconfig cho BusyBox trên Ubuntu là do thiếu thư viện ncurses. Để khắc phục, bạn cần cài đặt thư viện này. Hãy thử các bước sau:

Cài đặt thư viện ncurses:
```
sudo apt-get install libncurses5-dev libncursesw5-dev
```

Chạy lại lệnh make menuconfig:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

Nếu bạn vẫn gặp lỗi, hãy kiểm tra lại các bước cài đặt và cấu hình của BusyBox. Bạn có thể tham khảo thêm tài liệu hoặc hướng dẫn trực tuyến để đảm bảo mọi thứ được thiết lập đúng cách. Nếu cần thêm hỗ trợ, đừng ngần ngại hỏi nhé!


# 7. NFS
host pc:
```
sudo exportfs -rav
```
or
```
sudo exportfs -rv
```

BBB:
```
sudo mount 192.168.1.61:/srv/nfs/bin_code /home/debian/bin_files
sudo umount /home/debian/bin_files
```

# 8 Install and remove module
sudo rmmod pcd_device_setup.ko 
sudo insmod pcd_platform_driver.ko

# 9 Setup Static Ip address 
 
(00 or 01 ,..)
$ sudo nvim /etc/netplan/01-installer-config.yaml
![[Pasted image 20250302115237.png]]
```
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.32.95/24]
      gateway4: 192.168.32.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
$ sudo netplan try
$ sudo netplan apply
$ sudo reboot


