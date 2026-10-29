# Skill FigCraft untuk AI Agent

Skill agent yang mengajarkan coding agent cara mendesain dengan
[FigCraft](https://github.com/storytold/figcraft), aplikasi desain open-source offline bergaya Figma,
melalui server MCP bawaan-nya.

Skill dan server MCP adalah dua komponen terpisah, dan Anda membutuhkan keduanya:

| Komponen | Fungsinya | Lokasi |
|---|---|---|
| **Skill** (repo ini) | Instruksi: konvensi, alur kerja, hal-hal yang perlu diperhatikan | `~/.agents/skills/figcraft/` |
| **Server MCP** (disertakan bersama FigCraft) | Alat-alatnya: `create_node`, `export`, `screenshot`… | konfigurasi MCP agent Anda |

Tanpa skill, agent punya alat-alatnya tapi harus menebak konvensinya. Tanpa server MCP, agent tahu
apa yang harus dilakukan tapi hanya bisa memakai perintah `figcraft-cli` di command line.

## 1. Instal FigCraft

Unduh installer dari [rilis FigCraft](https://github.com/storytold/figcraft/releases),
atau bangun CLI dari source:

```
git clone https://github.com/storytold/figcraft
cd figcraft
cargo build --release -p figcraft-cli
```

Installer Windows menempatkan `figcraft-cli.exe` di `C:\Program Files\FigCraft\`.

## 2. Instal skill

Dengan CLI [`skills`](https://skills.sh/):

```
npx skills add <username-anda>/figcraft-skill -g
```

Atau salin `skills/figcraft/` ke folder skills yang dibaca agent Anda (untuk banyak agent, lokasinya
ada di `~/.agents/skills/`):

```powershell
Copy-Item -Recurse skills\figcraft $HOME\.agents\skills\figcraft
```

Cek dengan `npx skills ls -g`.

## 3. Daftarkan server MCP

Perintahnya selalu `figcraft-cli mcp`. Yang berbeda hanya tempat mendaftarkannya, tergantung agent
yang digunakan.

**Claude Code**

```
claude mcp add figcraft -- "C:\Program Files\FigCraft\figcraft-cli.exe" mcp
```

**Oh My Pi (OMP)**: jalankan `/mcp add`, pilih *User level*, lalu isi:

```
Name:      figcraft
Transport: stdio
Command:   C:\Program Files\FigCraft\figcraft-cli.exe
Arguments: mcp
```

**Agent yang membaca file JSON `mcpServers`**

```json
{
  "mcpServers": {
    "figcraft": {
      "command": "C:\\Program Files\\FigCraft\\figcraft-cli.exe",
      "args": ["mcp"]
    }
  }
}
```

Format konfigurasi berbeda antar agent (sebagian menggunakan array untuk `command`, Codex memakai
TOML), jadi cek dokumentasi MCP agent Anda jika format ini ditolak.

### Headless atau live

- **Headless** (konfigurasi di atas): agent bekerja pada dokumen di memori. Tidak ada yang tersimpan
  ke disk sampai di-save atau diekspor, jadi beritahu agent ke mana harus menyimpan.
- **Live**: jalankan aplikasi dengan `figcraft --control 7979`, dan daftarkan server dengan
  `args: ["mcp", "--attach", "7979"]`. Anda bisa melihat setiap perubahan di jendela aplikasi, dan
  beberapa agent bisa berbagi dokumen yang sama.

Untuk membatasi folder yang bisa dibaca dan ditulis agent, tambahkan
`--automation-read-root <dir>` dan `--automation-write-root <dir>` ke args.

## 4. Coba

> Gunakan FigCraft untuk mendesain layar utama mobile 375×812 dengan sapaan, kolom pencarian, dan
> tombol biru. Simpan ke `C:\designs\home.figcraft` dan ekspor PNG di sebelahnya.

## Memperbarui referensi

`skills/figcraft/references/mcp.md` adalah salinan dari `docs/mcp.md` milik FigCraft. File ini
berubah seiring perkembangan FigCraft. Untuk memperbaruinya:

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/storytold/figcraft/HEAD/docs/mcp.md `
  -OutFile skills\figcraft\references\mcp.md
```

Jika Anda mengubah heading di file tersebut, perbarui tabel referensi di bagian bawah `SKILL.md`.

## Kredit dan lisensi

- `skills/figcraft/references/mcp.md` disalin dari dokumentasi FigCraft, © 2026 ArtCraft Team dan
  kontributor FigCraft, yang dilisensikan ganda di bawah MIT atau Apache-2.0. Lihat
  [`LICENSE-MIT`](https://github.com/storytold/figcraft/blob/main/LICENSE-MIT),
  [`LICENSE-APACHE`](https://github.com/storytold/figcraft/blob/main/LICENSE-APACHE), dan
  [`NOTICE`](https://github.com/storytold/figcraft/blob/main/NOTICE) di upstream.
- Nama dan logo ArtCraft adalah merek dagang dari ArtCraft Team dan bukan bagian dari repo ini.
- Sisa repo ini (`SKILL.md` dan README ini) dilisensikan di bawah file lisensi yang Anda tambahkan.
- Ini adalah skill komunitas tidak resmi dan tidak berafiliasi dengan proyek FigCraft.