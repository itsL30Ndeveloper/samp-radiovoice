# **RADIOVOICE**
Indonesia | [English](https://github.com/itsL30Ndeveloper/samp-radiovoice/README.md

## Description
---------------------------------
**RADIOVOICE** - adalah Fitur tambahan untuk mengimplementasikan sistem komunikasi suara dalam bahasa _Pawno_ untuk server SA:MP dari sistem **SAMPVOICE**.

#### Dukungan versi
----------------------------------
* Client: SA:MP 0.3.7 (R1, R3)
* Server: SA:MP 0.3.7 (R2)

## Fitur
---------------------------------
* Transmisi suara terkontrol
* Kontrol mikrofon pemutar
* Mengikat aliran suara ke objek game
* Suara jarak jauh
* Frekuensi suara

## Instalasi
---------------------------------
Agar plugin berfungsi, itu harus diinstal oleh pemain dan di server. Ada bagian klien dan server dari plugin untuk ini.

#### Untuk pemain
---------------------------------
Pemain memiliki akses ke 2 opsi instalasi: otomatis (melalui penginstal) dan manual (melalui arsip).

##### Secara otomatis
---------------------------------
1. Untuk mengunduh penginstal, buka [halaman `rilis`](https://github.com/CyberMor/sampvoice/releases) dan pilih versi plugin yang diinginkan.
2. Setelah mengunduh, luncurkan installer dan pilih bahasa yang diinginkan untuk instalasi Anda, setelah itu installer akan secara otomatis menemukan folder GTA San Andreas Anda.
3. Jika direktori sudah benar, klik "OK" dan tunggu hingga instalasi selesai. Setelah penginstalan selesai, penginstal akanexit.

##### Secara manual
----------------------------------
1. Buka [halaman `rilis`](https://github.com/CyberMor/sampvoice/releases) dan unduh arsip dengan versi klien yang diinginkan.
2. Ekstrak arsip ke folder GTA San Andreas Anda.

#### Untuk pengembang
----------------------------------
1. Unduh dari [halaman `rilis`](https://github.com/CyberMor/sampvoice/releases) versi plugin yang diinginkan untuk platform Anda.
2. Buka paket arsip ke direktori root server.
3. Tambahkan ke file konfigurasi server *server.cfg* baris *"plugins sampvoice"* untuk *Win32* dan *"plugins sampvoice.so"* untuk *Linux x86*. **(Jika Anda memiliki plugin Pawn.RakNet pastikan untuk menempatkan SampVoice setelahnya)**
## Penggunaan
----------------------------------
Untuk mulai menggunakan plugin, baca dokumentasi yang disertakan dengan sisi server. Untuk melakukannya, buka file *sampvoice.chm* menggunakan referensi Windows. **(Jika dokumentasi tidak terbuka, klik kanan pada file dokumentasi, lalu Properties -> Unblock -> OK)**

Untuk mulai menggunakan fungsionalitas plugin, sertakan file header:
```php
#include <sampvoice>
#include <sscanf2>
#include <a_samp>
#include <zcmd>
```
#### Referensi cepat
----------------------------------
Perlu Anda ketahui bahwa plugin menggunakan tipe dan sistem konstannya sendiri. Terlepas dari kenyataan bahwa ini hanya pembungkus tipe dasar Pion, ini membantu untuk menavigasi jenis plugin itu sendiri dan tidak membingungkan pointer.

Untuk mengalihkan lalu lintas audio dari pemutar A ke pemutar B, Anda perlu membuat streaming audio (misalnya, streaming global, menggunakan **SvCreateGStream**), lalu melampirkannya ke streaming pemutar A sebagai speaker (menggunakan **SvAttachSpeakerToStream**), setelah itu lampirkan ke aliran pemain B sebagai pendengar (menggunakan **SvAttachListenerToStream**). Selesai. Sekarang, saat mikrofon pemain A diaktifkan (misalnya, dengan fungsi **SvStartRecord**), lalu lintas audionya akan ditransmisikan dan kemudian didengar oleh pemain B.

Aliran suara cukup berguna. Mereka dapat divisualisasikan menggunakan contoh Discord:
* Aliran adalah analog dari ruangan (atau saluran).
* Pembicara adalah peserta dalam ruangan dengan bisu tetapi mikrofon menyala.
* Pendengar adalah peserta dalam ruangan dengan mikrofon mereka bisu tapi bisu.

Pemain dapat menjadi pembicara dan pendengar pada saat yang bersamaan. Dalam hal ini, lalu lintas audio pemain tidak akan diteruskan kepadanya.

#### Contoh
----------------------------------
Mari kita lihat beberapa fitur plugin dengan contoh praktis. Di bawah ini kami akan membuat server yang akan mengikat semua pemain yang terhubung ke aliran global, dan juga membuat aliran lokal untuk setiap pemain. Dengan demikian, pemain akan dapat berkomunikasi melalui obrolan global (terdengar secara merata di titik mana pun di peta) dan lokal (hanya terdengar di dekat pemain).
```php
#include <a_samp>
#include <zcmd>
#include <sampvoice>
#include <sscanf2>

#define MAX_RADIOS 9999 //maximum radio frequency channel

new SV_GSTREAM:RadioChannel[MAX_RADIOS] = SV_NULL; 
new FreqNumber[MAX_PLAYERS]; //frequency number
new SV_GSTREAM:gstream = SV_NULL;
new SV_LSTREAM:lstream[MAX_PLAYERS] = { SV_NULL, ... };

/*
    The public OnPlayerActivationKeyPress and OnPlayerActivationKeyRelease
    are needed in order to redirect the player's audio traffic to the
    corresponding streams when the corresponding keys are pressed.
*/

public SV_VOID:OnPlayerActivationKeyPress(SV_UINT:playerid, SV_UINT:keyid) 
{
    // Attach player to local stream as speaker if 'B' key is pressed
    if (keyid == 0x42 && lstream[playerid]) SvAttachSpeakerToStream(lstream[playerid], playerid);
    // Attach the player to the global stream as a speaker if the 'Z' key is pressed
    if (keyid == 0x5A && gstream) SvAttachSpeakerToStream(gstream, playerid);
    // Attach player to radio stream as speaker when 'B' button is pressed
    if(keyid == 0x42 && FreqNumber[playerid] >= 1) SvAttachSpeakerToStream(RadioChannel[FreqNumber[playerid]], playerid);
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public SV_VOID:OnPlayerActivationKeyRelease(SV_UINT:playerid, SV_UINT:keyid)
{
    // Detach the player from the local stream if the 'B' key is released
    if (keyid == 0x42 && lstream[playerid]) SvDetachSpeakerFromStream(lstream[playerid], playerid);
    // Detach the player from the global stream if the 'Z' key is released
    if (keyid == 0x5A && gstream) SvDetachSpeakerFromStream(gstream, playerid);
    // Detach the player from the radio stream if the 'B' key is released
    if(keyid == 0x42 && FreqNumber[playerid] >= 1) SvDetachSpeakerFromStream(RadioChannel[FreqNumber[playerid]], playerid);
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public OnPlayerConnect(playerid)
{
    // Checking for plugin availability
    if (SvGetVersion(playerid) == SV_NULL)
    {
        SendClientMessage(playerid, -1, "Could not find plugin sampvoice.");
    }
    // Checking for a microphone
    else if (SvHasMicro(playerid) == SV_FALSE)
    {
        SendClientMessage(playerid, -1, "The microphone could not be found.");
    }
    // Create a local stream with an audibility distance of 40.0, an unlimited number of listeners
    // and the name 'Local' (the name 'Local' will be displayed in red in the players' speakerlist)
    else if ((lstream[playerid] = SvCreateDLStreamAtPlayer(40.0, SV_INFINITY, playerid, 0xff0000ff, "Local")))
    {
        SendClientMessage(playerid, -1, "Press Z to talk to global chat and B to talk to local chat.");

        // Attach the player to the global stream as a listener
        if (gstream) SvAttachListenerToStream(gstream, playerid);

        // Assign microphone activation keys to the player
        SvAddKey(playerid, 0x42);
        SvAddKey(playerid, 0x5A);
    }
}

public OnPlayerDisconnect(playerid, reason)
{
    // Removing the player's local stream after disconnecting
    if (lstream[playerid])
    {
        SvDeleteStream(lstream[playerid]);
        lstream[playerid] = SV_NULL;
    }
}

public OnGameModeInit()
{
    // Uncomment the line to enable debug mode
    // SvDebug(SV_TRUE);

    gstream = SvCreateGStream(0xffff0000, "Global");
}

public OnGameModeExit()
{
    if (gstream) SvDeleteStream(gstream);
}

CMD:setfreq(playerid, params[])
{
	new id;
	if(sscanf(params, "d", id)) return SendClientMessage(playerid, -1, "Use: /setfreq [frequensi] | 0 = turn off");

	FreqNumber[playerid] = id;

	if(RadioChannel[FreqNumber[id]] == SV_NULL)
	{
		RadioChannel[FreqNumber[id]] = SvCreateGStream(0xFF00FFFF, "Radio");
		SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
 	}
 	else
 	{
    	SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
	}
	return 1;
}

```

## Kompilasi
----------------------------------
Kompilasi plugin untuk platform *Win32* dan *Linux x86*.

Di bawah ini adalah instruksi lebih lanjut:

Kloning repositori ke komputer Anda dan buka direktori plugin:
```sh
git clone https://github.com/CyberMor/sampvoice.git
cd sampvoice
```

### Windows (Klien/Server)
----------------------------------
Untuk mengompilasi sisi klien plugin, Anda memerlukan *DirectX SDK*. Secara default, bagian klien dikompilasi untuk versi **SA: MP 0.3.7 (R1)**, tetapi Anda juga dapat secara eksplisit memberi tahu compiler versi untuk build menggunakan **SAMP_R1** dan **SAMP_R3** makro. Untuk membangun bagian klien dan server dari plugin untuk platform *Win32*, buka proyek *sampvoice.sln* di MS Visual Studio 2019 dan kompilasi:
> Bangun -> Bangun Solusi (F7)

### Linux (Server)
----------------------------------
Untuk membangun bagian server dari plugin untuk platform *Linux x86*, ikuti petunjuk berikut:
```sh
cd server
make
```
