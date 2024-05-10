#mengimpor  modul- modul yang akan digunakan
import tkinter as tk
from tkinter import messagebox, simpledialog
import datetime
import random

#merepresentasikan setiap tiket parkir secara terpisah dalam program
class Parkir:
    def __init__(self, jenis_kendaraan, waktu_masuk):
        self.jenis_kendaraan = jenis_kendaraan
        self.waktu_masuk = waktu_masuk
        self.waktu_keluar = None
        self.biaya = 0
        self.random_code = None

#menghitung biaya parkir
    def hitung_biaya(self, waktu_keluar):
        self.waktu_keluar = waktu_keluar
        durasi = (waktu_keluar.hour) - (self.waktu_masuk.hour )
        if self.jenis_kendaraan == "mobil":
            self.biaya = durasi * 5000
        elif self.jenis_kendaraan == "motor":
            self.biaya = durasi * 2000

#membuat code acak
    def generate_random_code(self):
        self.random_code = ''.join(random.choices('0123ABCDEFGHIJKLMNOPQRSTUVWXYZ', k=3))
        return self.random_code

#menyimpan riwayat parkir / kapisitas parkir
history = {}
kapasitas_parkir = {
    'GEDUNG LAB': {'motor': 1, 'mobil': 5},
    'GEDUNG DEPARTEMEN': {'motor': 1, 'mobil': 5},
    'GEDUNG GANDUNG GINDANG': {'motor': 1, 'mobil': 5},
    'GEDUNG KESENIAN': {'motor': 20, 'mobil': 10},
    'GEDUNG OLAHRAGA': {'motor': 20, 'mobil': 10},
    'GEDUNG KITA': {'motor': 20, 'mobil': 10},
    'GEDUNG MUSIK': {'motor': 20, 'mobil': 10}
}

#mengaatur tampilan awal aplikasi yang berisi beberapa menu
class ParkingSystemApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistem Manajemen Parkir")
        self.geometry("400x300")

        self.frames = {}

        for F in (MainMenu, MasukParkir, KeluarParkir, KapasitasParkir):
            frame = F(self)
            self.frames[F] = frame

        self.show_frame(MainMenu)

#menampilkan frame yang diinginkan
    def show_frame(self, cont):
        for frame in self.frames.values():
            frame.pack_forget()
        frame = self.frames[cont]
        frame.pack(expand=True, fill="both")

#membuat tampilan awal menu utama
class MainMenu(tk.Frame):
    def __init__(self, parent):
        super().__init__(parent)

        label = tk.Label(self, text="Menu Utama", font=('Helvetica', 18))
        label.pack(pady=20)

        button_masuk = tk.Button(self, text="Masuk ke Parkiran", command=lambda: parent.show_frame(MasukParkir))
        button_masuk.pack()

        button_keluar = tk.Button(self, text="Keluar dari Parkiran", command=lambda: parent.show_frame(KeluarParkir))
        button_keluar.pack()

        button_kapasitas = tk.Button(self, text="Lihat Kapasitas Tempat Parkir", command=lambda: parent.show_frame(KapasitasParkir))
        button_kapasitas.pack()

#membuat tampilan awal menu masuk ke parkiran
class MasukParkir(tk.Frame):
    def __init__(self, parent):
        super().__init__(parent)

        label = tk.Label(self, text="Masuk ke Parkiran", font=('Helvetica', 18))
        label.pack(pady=20)

        label_gedung = tk.Label(self, text="Pilih lokasi gedung:")
        label_gedung.pack()

        self.lokasi_gedung_var = tk.StringVar(self)
        self.lokasi_gedung_var.set("A")  # Default value
        gedung_option = tk.OptionMenu(self, self.lokasi_gedung_var, *kapasitas_parkir.keys())
        gedung_option.pack()

        label_kendaraan = tk.Label(self, text="Pilih jenis kendaraan:")
        label_kendaraan.pack()

        self.jenis_kendaraan_var = tk.StringVar(self)
        self.jenis_kendaraan_var.set("mobil")  # Default value
        kendaraan_option = tk.OptionMenu(self, self.jenis_kendaraan_var, "mobil", "motor")
        kendaraan_option.pack()

        button_masuk = tk.Button(self, text="Masuk", command=self.masuk_parkir)
        button_masuk.pack(pady=10)

        button_kembali = tk.Button(self, text="Kembali ke Menu Utama", command=lambda: parent.show_frame(MainMenu))
        button_kembali.pack(pady=10)

#fungsi agar dapat beralih/masuk ke menu "masuk ke parkiran"
    def masuk_parkir(self):
        lokasi_gedung = self.lokasi_gedung_var.get()
        jenis_kendaraan = self.jenis_kendaraan_var.get()

        if kapasitas_parkir[lokasi_gedung][jenis_kendaraan] == 0:
            messagebox.showerror("Error", "Maaf, tempat parkir untuk jenis kendaraan tersebut sudah penuh di gedung ini.")
            return

        waktu_masuk = datetime.datetime.now()
        tiket_parkir = Parkir(jenis_kendaraan, waktu_masuk)
        random_code = tiket_parkir.generate_random_code()
        history[random_code] = {
            'jenis_kendaraan': jenis_kendaraan,
            'lokasi_gedung': lokasi_gedung,
            'waktu_masuk': waktu_masuk
        }
        kapasitas_parkir[lokasi_gedung][jenis_kendaraan] -= 1
        messagebox.showinfo("Info", f"Selamat! Anda berhasil masuk ke gedung {lokasi_gedung}. Kode acak Anda adalah {random_code}")

#membuat tampilan menu keluar parkiran
class KeluarParkir(tk.Frame):
    def __init__(self, parent):
        super().__init__(parent)

        label = tk.Label(self, text="Keluar dari Parkiran", font=('Helvetica', 18))
        label.pack(pady=20)

        label_kode = tk.Label(self, text="Masukkan kode acak:")
        label_kode.pack()

        self.kode_var = tk.StringVar(self)
        entry_kode = tk.Entry(self, textvariable=self.kode_var)
        entry_kode.pack()

        button_keluar = tk.Button(self, text="Keluar", command=self.keluar_parkir)
        button_keluar.pack(pady=10)

        button_kembali = tk.Button(self, text="Kembali ke Menu Utama", command=lambda: parent.show_frame(MainMenu))
        button_kembali.pack(pady=10)

# fungsi agar dapat beralih/masuk ke menu "keluar parkiran"
    def keluar_parkir(self):
        code = self.kode_var.get()
        if code not in history:
            messagebox.showerror("Error", "Kode acak tidak valid. Silakan coba lagi.")
            return

        jenis_kendaraan = history[code]['jenis_kendaraan']
        lokasi_gedung = history[code]['lokasi_gedung']
        waktu_masuk = history[code]['waktu_masuk']

        waktu_keluar = datetime.datetime.now().time()
        kapasitas_parkir[lokasi_gedung][jenis_kendaraan] += 1
        durasi = (waktu_keluar.hour - waktu_masuk.hour)

        tiket_parkir = Parkir(jenis_kendaraan, waktu_masuk)
        tiket_parkir.hitung_biaya(waktu_keluar)
        messagebox.showinfo("TIKET PARKIR", f"Waktu Masuk Anda Adalah: {waktu_masuk} \nWaktu Keluar Anda Adalah: {waktu_keluar} \nDurasi parkir Anda adalah: {durasi} jam \nJenis Kendaraan Anda: {jenis_kendaraan} \nBiaya Parkir Anda adalah Rp. {tiket_parkir.biaya}")

#membuat tampilan menu kapasitas parkir
class KapasitasParkir(tk.Frame):
    def __init__(self, parent):
        super().__init__(parent)

        label = tk.Label(self, text="Lihat Kapasitas Tempat Parkir", font=('Helvetica', 18))
        label.pack(pady=20)

        label_gedung = tk.Label(self, text="Masukkan lokasi gedung:")
        label_gedung.pack()

        self.gedung_var = tk.StringVar(self)
        self.gedung_var.set("A")  # Default value
        gedung_option = tk.OptionMenu(self, self.gedung_var, *kapasitas_parkir.keys())
        gedung_option.pack()

        button_lihat = tk.Button(self, text="Lihat", command=self.lihat_kapasitas)
        button_lihat.pack(pady=10)

        button_kembali = tk.Button(self, text="Kembali ke Menu Utama", command=lambda: parent.show_frame(MainMenu))
        button_kembali.pack(pady=10)

# fungsi agar dapat beralih/masuk ke menu "kapasitas parkir"
    def lihat_kapasitas(self):
        lokasi_gedung = self.gedung_var.get()
        messagebox.showinfo("Kapasitas Tempat Parkir", f"Kapasitas tempat parkir di gedung {lokasi_gedung}:\n- Motor: {kapasitas_parkir[lokasi_gedung]['motor']} kendaraan\n- Mobil: {kapasitas_parkir[lokasi_gedung]['mobil']} kendaraan")

#agar semua fungsi dapat dijalnkan/ menjalankan aplikasi
if __name__ == "__main__":
    app = ParkingSystemApp()
    app.mainloop()
