# Reserve Enginer
## Description
- objdump v.2.30
- bahasa C
- menapilkan text 'b01ler up\n'

## Configuration
` set disasembly-flavor intel `
konfigurasi ini bertujuan agar aplikasi gdb dapat menampilkan syntax intel.

## 0x1 - Menganalisa printf text
Buat sebuah program sederhana menggunakan bahasa C, yang mana program tersebut menampikan sebuah text string `b01ler up`.

```
#include <stdio.h>

int main(int argc, char** argv) {
        printf("b01ler up!\n");
        return 0;
}

```

## 0x1 - Compile
compile program dengan perintah
` gcc main.c -o b01ler `

## x01 - bongkar kode program menggunakan objdump
dalam disassemble program kita dunakan option `-D` dan `-M`. `-D` berarti melakukan disassemble untuk semua bagian program dan `-M` adalah format ouputnya dalam bentuk apa misal `intel`. by default unix menggunakan format `AT&T` yang sulit untuk di baca.
perintah yang akan di gunakan adalah seperti berikut `objdump -D -M intel b01ler`.

akan muncul semua bagian dari program, coba cari bagian main. kurang lebih seperti berikut :
```
0000000000000605 <main>:
 605:	55                   	push   rbp
 606:	48 89 e5             	mov    rbp,rsp
 609:	48 83 ec 10          	sub    rsp,0x10
 60d:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
 610:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
 614:	48 8d 3d 99 00 00 00 	lea    rdi,[rip+0x99]        # 6b4 <_IO_stdin_used+0x4>
 61b:	e8 f0 fe ff ff       	call   510 <puts@plt>
 620:	b8 00 00 00 00       	mov    eax,0x0
 625:	c9                   	leave
 626:	c3                   	ret
 627:	66 0f 1f 84 00 00 00 	nop    WORD PTR [rax+rax*1+0x0]
 62e:	00 00
```

bagian `0000000000000605 <main>:` ini maksudnya fungsi main beralamat di `0000000000000605`.
alamat `605`, `606` adalah prolog program.
pada bagian `614:	48 8d 3d 99 00 00 00 	lea    rdi,[rip+0x99]        # 6b4 <_IO_stdin_used+0x4>`, berhubungan dengan pemanggilan fungsi printf / input-output. terus dimana lokasi string `b01ler up` yang ada padaprogram ??.
perhatikan baris tersebut ada intruksi `lea`, intruksi ini memberitahu pada si CPU untuk memuat alamat `0x99 byte` yang ada di alamat `#6b4`.

sekarang cari alamat `#6b4`, outpunya seperti ini :
```
Disassembly of section .rodata:
00000000000006b0 <_IO_stdin_used>:
 6b0:	01 00                	add    DWORD PTR [rax],eax
 6b2:	02 00                	add    al,BYTE PTR [rax]
 6b4:	62                   	(bad)
 6b5:	30 31                	xor    BYTE PTR [rcx],dh
 6b7:	6c                   	ins    BYTE PTR es:[rdi],dx
 6b8:	65 72 20             	gs jb  6db <__GNU_EH_FRAME_HDR+0x1b>
 6bb:	75 70                	jne    72d <__GNU_EH_FRAME_HDR+0x6d>
 6bd:	21 00                	and    DWORD PTR [rax],eax
```

dan ternyata alamat `#6b4` adalah alamat untuk bagian program data / `.rodata`. jika kita lihat alamat `0x6b4` dan seterusnya kita lihat byte- byte dalam format ASCII yang mempresentasikan text string.
untuk membuktikannya kita coba pake python untuk melakukan convert dari ASCII ke string.
``` >>> [chr(x) for x in [0x62, 0x30,0x31,0x6c,0x65,0x72,0x20,0x75,0x70,0x21,0x00]]
['b', '0', '1', 'l', 'e', 'r', ' ', 'u', 'p', '!', '\x00']
```
benar bukan, ada cara lain juga untuk membaca ada text string atau tidak dalam program, menggunakan perintah `strings b01ler`. tapi hal ini bisa jadi berbahaya karena bisa jadi ketika menjalankan `string b01ler` ada exploit yang jalan tanpa kita tahu (https://lcamtuf.blogspot.com/2014/10/psa-dont-run-strings-on-untrusted-files.html)

dan terakhir dari kode `return 0` setelah pemanggilan string.
```
 620:	b8 00 00 00 00       	mov    eax,0x0
 625:	c9                   	leave
 626:	c3                   	ret
```
intruksi eax ke 0 mendefinisikan register AX digunakan untuk returns

`CMIIW - this my journey learn about RE`
