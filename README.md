# SoalShift_modul4_A08
TUGAS SOAL SHIFT SISTEM OPERASI kelompok A08

# Nomor 1 

Semua nama file dan folder harus terenkripsi
Enkripsi yang Atta inginkan sangat sederhana, yaitu Caesar cipher. Namun, Kusuma mengatakan enkripsi tersebut sangat mudah dipecahkan. Dia menyarankan untuk character list diekspansi,tidak hanya alfabet, dan diacak. Berikut character list yang dipakai:

qE1~ YMUR2"`hNIdPzi%^t@(Ao:=CQ,nx4S[7mHFye#aT6+v)DfKL$r?bkOGB>}!9_wV']jcp5JZ&Xl|\8s;g<{3.u*W-0`

Misalkan ada file bernama “halo” di dalam folder “INI_FOLDER”, dan key yang dipakai adalah 17, maka:
“INI_FOLDER/halo” saat belum di-mount maka akan bernama “n,nsbZ]wio/QBE#”, saat telah di-mount maka akan otomatis terdekripsi kembali menjadi “INI_FOLDER/halo” .
Perhatian: Karakter ‘/’ adalah karakter ilegal dalam penamaan file atau folder dalam *NIX, maka dari itu dapat diabaikan


- fungsi untuk mengenkripsi file sesuai aturan pada soal
```
void enkripsi(char* kata)
{
    char dasar[100] = "qE1~ YMUR2\"`hNIdPzi%^t@(Ao:=CQ,nx4S[7mHFye#aT6+v)DfKL$r?bkOGB>}!9_wV']jcp5JZ&Xl|\\8s;g<{3.u*W-0";
    for(int i=0; i<strlen(kata); i++){
        for(int j = 0 ; j<strlen(dasar); j++){
            if(kata[i] == dasar[j]){
                int indeks_baru = (j+17) % 94;
                kata[i] = dasar[indeks_baru];
                break;
            }
        }
    }
}
```

- fungsi untuk dekripsi file
```
void dekripsi(char* kata)
{
    char dasar[100] = "qE1~ YMUR2\"`hNIdPzi%^t@(Ao:=CQ,nx4S[7mHFye#aT6+v)DfKL$r?bkOGB>}!9_wV']jcp5JZ&Xl|\\8s;g<{3.u*W-0";
    for(int i=0; i<strlen(kata); i++){
        for(int j = 0 ; j<strlen(dasar); j++){
            if(kata[i] == dasar[j]){
                int indeks_baru = (j+(94-17)) % 94;
                kata[i] = dasar[indeks_baru];
                break;
            }
        }
    }
}
```

- fungsi `xmp_getattr` berfungsi sama dengan stat yaitu untuk mendapatkan semua atribut file/direktori.
- Dalam fungsi ini kita mengenkripsi `*path` dan diletakkan kedalam direktori yg disimpan di variabel `dirpath`. Jadi fusenya ada di `dirpath` itu.
```
static int xmp_getattr(const char *path, struct stat *stbuf) //path = filemiris.txt
{
    int res;
    ///
	char fpath[1000];
	char sementara[1000];
    sprintf(sementara,"%s",path); 

    enkripsi(sementara); //kalo langsung enk path error
	sprintf(fpath, "%s%s",dirpath,sementara);
    ///
	res = lstat(fpath, stbuf);

	if (res == -1)
		return -errno;

	return 0;
}
```

# Soal 3

Sebelum diterapkannya file system ini, Atta pernah diserang oleh hacker LAPTOP_RUSAK yang menanamkan user bernama “chipset” dan “ic_controller” serta group “rusak” yang tidak bisa dihapus. Karena paranoid, Atta menerapkan aturan pada file system ini untuk menghapus “file bahaya” yang memiliki spesifikasi:
Owner Name 	: ‘chipset’ atau ‘ic_controller’
Group Name	: ‘rusak’
Tidak dapat dibaca
Jika ditemukan file dengan spesifikasi tersebut ketika membuka direktori, Atta akan menyimpan nama file, group ID, owner ID, dan waktu terakhir diakses dalam file “filemiris.txt” (format waktu bebas, namun harus memiliki jam menit detik dan tanggal) lalu menghapus “file bahaya” tersebut untuk mencegah serangan lanjutan dari LAPTOP_RUSAK.

- Dalam code dibawah karena file yg diakses ketika  kita melakukan fuse adalah yang ada di path `dirpath` dan yang ada di `dirpath` itu adalah yang di enkripsi maka setiap kita melakukan operasi seperti ls atau stat dll harus di enkrip dulu. Karena nama yang kita masukan kan tidak ada di tempat yg sebenarnya kita akses.
```
	char fpath[1000];

    char sementara[1000];
    sprintf(sementara,"%s",path);

    enkripsi(sementara);

	sprintf(fpath, "%s%s",dirpath,sementara);
	
	int res = 0;
```

- Di fungsi ini dia membuka direktori kemudian membaca file/direktori didalam direktori tersebut. Kemudian seluruh informasinya disimpan di `de` yang bentuknya `struct`. Kemudian karena cara kerjanya seperti ls makaa yang diambil hanya `d_name` nya saja. 
```
	while ((de = readdir(dp)) != NULL) {
		struct stat info;
        char cek[10000];
        strcpy(cek,fpath);
        strcat(cek,de->d_name);

        char file[1000];
 ```
 
 - Kemudian di sini `.` sm `..` tidak di dekripsi lagi karena `ls .` itu untuk mengecek isi direktori direktori itu sendiri dan `ls ..` untuk direktori yang menyimpanya yang lebih belakang lagi. Terus kalau selain itu didekripsi karena yang kita akses kan file yg fuse yang terenkripsi kalau ls kan mintanya nampilinya nama asli bukan yng terenkripsi jadi nama yang dalam bentuk enkripsi itu harus didekripsi agar kembali namanya spert biasa.
 
 ```
		//
		if(strcmp(de->d_name,".")!=0 && strcmp(de->d_name,"..")!=0) dekripsi(de->d_name);
		//
```

- disini info menyimpan semua yang didapat dari perintah stat. Tapi yang kita ambil hanya `st_uid` dan `st_gid` untuk user dan group nya.
```
        stat(cek,&info);
        struct passwd *user;
        user = getpwuid(info.st_uid); //mengambil user dari stat info.uid
        struct group *grup;
        grup = getgrgid(info.st_gid);//mengambil grup dari stat info.uid
```

- Nah disini mulai dicek apakah usernya `chipset` atau `ic_contoller` dan group nya `rusak` dan file tidak bisa dibaca (`R_OK ==0`)
- Kalau iya maka akan dibuat filemiris.txt trus filemiris.txt kan juga masuk ke folder fusenya. Maka dienkripsi juga dan ditaruh di fuse. Kemudian, file tersebut di catat dan dimasukan ke filemiris.txt dengan format nama dan waktu penghapusan. Kemudian dihapus. Nah kalo filemiris.txt nya udah ada maka akan di append saja dengan `a+` .

```
        if( (strcmp(user->pw_name,"chipset") == 0 || strcmp(user->pw_name,"ic_controller") == 0) && strcmp(grup->gr_name,"rusak") == 0){
          if((info.st_mode & R_OK)==0){ //tdkbisa dibaca
              char txt[10000] = "/filemiris.txt";
              enkripsi(txt);
              char pathtxt[100000];
              sprintf(pathtxt,"%s%s",dirpath,txt);

              FILE *filetxt;
              filetxt = fopen(pathtxt,"a+");

              char waktu[21];
			  time_t now = time(NULL);
			  strftime(waktu, 20, "%H:%M:%S %Y-%m-%d", localtime(&now));//mengubah format waktu jd string
			
			  char isi[1000]; //apa yg mau ditulis di filemiris.txt
              strcpy(isi,de->d_name);
              strcat(isi,"_");
              char iduser[1000];
              sprintf(iduser,"%d_%d",user->pw_uid,grup->gr_gid);
              strcat(isi,iduser);
              strcat(isi,"_");
              strcat(isi,waktu);

              fprintf(filetxt,"%s\n",isi); //untuk print di filemiris
              fclose(filetxt);
              remove(cek);
          }
        }
 ```
- Kalau ternyata bukan sesuai kriteria maka ditampilkan saja. ls sperti biasa tidak dihapus dan dicatat
 ```
        else{ // tdk di remove
            struct stat st;
		    memset(&st, 0, sizeof(st));
		    st.st_ino = de->d_ino;
		    st.st_mode = de->d_type << 12;

            strcpy(file,de->d_name);
		    res = (filler(buf, file, &st, 0));
		    	if(res!=0) break;
        }
	}

	closedir(dp);
	return 0;
}
```





Side note : Terimaksih mas, sudah mengajari saya yang lemot ini dengan sabar. Saya pasti banyak salaah, maaf ya mas. Terimakasiiih hehe.
