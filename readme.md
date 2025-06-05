<h1 style="text-align:center">🎯 Action Ring</h1>

<p align="center">
  GTK4 & libadwaita tabanlı, hızlı erişim ve komut çalıştırma uygulaması.<br>
  Webcord ve Zapzap ile uyumlu.<br>
</p>

---

<h2 style="text-align:center">🚀 Özellikler</h2>

- Kişiselleştirilebilir tuş ve komut yönetimi  
- Webcord ve Zapzap ile entegre çalışma  
- Flatpak ile kolay kurulum ve güncelleme  
- Hafif ve hızlı performans  

---

<h2 style="text-align:center">🛠️ Gereksinimler</h2>

- Python 3  
- PyGObject (Python GTK bindings)  
- GTK 4 ve libadwaita (sistem kütüphaneleri)  
- <b>Webcord ve Zapzap uygulamalarının Flatpak ile kurulumu (zorunlu)</b>  

<p align="center">
  <i>Not:</i> Bu sürümü kullanmak için <b>Webcord</b> ve <b>Zapzap</b>'ın Flatpak versiyonlarının sisteminizde kurulu olması gerekmektedir.
</p>

---

<h2 style="text-align:center">📦 Kurulum</h2>

### Sistem paketleri ve Python bağımlılıkları:

```bash
sudo apt update
sudo apt install python3 python3-gi python3-gi-cairo gir1.2-gtk-4.0 gir1.2-adw-1
pip install -r requirements.txt
