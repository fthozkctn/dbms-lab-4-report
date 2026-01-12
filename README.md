# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [ ]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [X]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [ ]  LRU / CLOCK gibi algoritmaları
- [ ]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki](https://youtu.be/_xTmcG0B5hk) 

---

# Sistem Programlama ve Veri Yapıları Perspektifiyle Veritabanı Performans Analizi

1. Giriş
Veritabanı Yönetim Sistemleri (VTYS), veriyi sadece depolayan değil, aynı zamanda veriye en düşük maliyetle erişimi optimize eden karmaşık yazılım mimarileridir. Bu rapor kapsamında, endüstri standardı olan MSSQL veritabanı motoru ve Northwind örnek veri seti kullanılarak; veritabanı performansını doğrudan etkileyen sistem programlama prensipleri ve veri yapıları analiz edilmiştir. Analizimiz; disk erişim stratejileri, bellek hiyerarşisi yönetimi ve indeksleme algoritmaları olmak üzere üç temel sütun üzerine inşa edilmiştir.
2. Sistem Perspektifi: Sayfa (Page) Yapısı ve Disk I/O Optimizasyonu
Modern işletim sistemlerinde disk erişimi, bayt bazlı değil blok bazlı gerçekleşir. MSSQL mimarisi, bu donanım gerçekliğine uyum sağlamak amacıyla verileri 8 KB’lık "Page" (Sayfa) adı verilen mantıksal birimlerde saklar.Sistem programlama perspektifinden bakıldığında, disk kafasının (HDD için) veya kontrolcünün (SSD için) bir veriye ulaşması "I/O Latency" (Gecikme) yaratır. Eğer veritabanı her bir satır için ayrı bir disk isteği (I/O Request) gönderseydi, sistem "I/O Bound" durumuna düşerek kilitlenirdi. MSSQL, Northwind veritabanındaki Orders tablosundan bir kayıt okurken, o kaydın bulunduğu 8 KB'lık sayfanın tamamını tek bir atomik işlemle okur.Bu yaklaşım, "Spatial Locality" (Mekansal Yerellik) prensibinden faydalanır. Yani, bir kayda erişen kullanıcının muhtemelen o kayda komşu olan verilere de erişeceği varsayılır. Bu sayede, rastgele disk erişimi (Random I/O) minimize edilerek ardışık okuma (Sequential I/O) avantajı kazanılır ve toplam sistem throughput değeri artırılmış olur.
3. Bellek Yönetimi: Buffer Pool ve Veri Önbellekleme (Caching)
Veritabanı performansındaki en büyük darboğaz disk erişim hızıdır. RAM ile disk arasındaki hız farkı milisaniyeler ile nanosaniyeler arasındaki uçurum kadardır. MSSQL, bu farkı kapatmak için Buffer Pool adı verilen gelişmiş bir bellek yönetim mekanizması kullanır.Diskten okunan her 8 KB'lık sayfa, doğrudan RAM üzerindeki bu tampon bölgeye kopyalanır. Veritabanı motoru, bir veri talebi geldiğinde önce disk yerine bu bellek alanını kontrol eder (Buffer Hit). Northwind veritabanı üzerinde çalışırken kullanılan sys.dm_os_buffer_descriptors gibi dinamik yönetim görünümleri (DMV), hangi tabloların hangi oranda bellekte tutulduğunu şeffaf bir şekilde ortaya koymaktadır.Buffer Pool yönetimi, işletim sistemindeki "Paging" mekanizmasına benzer bir mantıkla çalışır. Bellek dolduğunda hangi sayfanın dışarı atılacağına karar veren LRU (Least Recently Used) veya MSSQL özelindeki gelişmiş saat algoritmaları, en sık kullanılan Northwind verilerinin (örneğin sürekli sorgulanan Products tablosu) her zaman RAM'de kalmasını sağlar. Bu, disk I/O operasyonlarını dramatik bir şekilde azaltarak sorgu yanıt sürelerini optimize eder.
4. Veri Yapıları: B+ Tree ve Hiyerarşik İndeksleme
Veritabanı performansının veri yapıları ayağını ise indeksleme algoritmaları oluşturur. MSSQL, verileri fiziksel olarak organize etmek ve arama maliyetini düşürmek için B+ Tree (Balanced Tree) veri yapısını kullanır.Northwind Orders tablosundaki OrderID alanı üzerinde tanımlı olan Clustered Index, tablonun kendisinin disk üzerinde bu B+ Tree yapısında dizilmesini sağlar. B+ Tree yapısının "Balanced" (Dengeli) olması, veritabanındaki kayıt sayısı milyonlara ulaşsa bile, aranan bir kayda sadece 3-4 adımda (Root -> Intermediate -> Leaf) ulaşılmasını garanti eder.Veri yapıları perspektifiyle bu durum, arama karmaşıklığını $O(n)$ (Linear Scan) düzeyinden $O(\log n)$ (Binary/Tree Search) düzeyine indirger. Ayrıca B+ Tree'nin yaprak düğümlerinin (Leaf Nodes) birbirine bağlı listeler (Linked List) şeklinde bağlanmış olması, "Range Scan" dediğimiz belirli tarih aralığındaki siparişlerin getirilmesi gibi işlemlerde disk üzerinde ileri yönlü çok hızlı bir tarama yapılabilmesine olanak tanır.
6. Sonuç
Yapılan incelemeler sonucunda görülmüştür ki; veritabanı performansı sadece yazılan SQL sorgusunun kalitesine değil, aynı zamanda alttaki donanım ve işletim sistemi kaynaklarının ne kadar efektif kullanıldığına bağlıdır. MSSQL'in 8 KB'lık sayfa yapısı disk darboğazını aşmayı, Buffer Pool mekanizması bellek hiyerarşisini verimli kullanmayı, B+ Tree yapısı ise matematiksel olarak en hızlı erişim yolunu sunmayı amaçlar. Northwind veri seti üzerinde yapılan testler, bu teorik altyapının pratik uygulamadaki başarısını kanıtlar niteliktedir.

## VT Üzerinde Gösterilen Kaynak Kodları

MSSQL Sayfa (Page) Yapısı Mimarisi [Linki](https://learn.microsoft.com/en-us/sql/relational-databases/pages-and-extents-architecture-guide?view=sql-server-ver16) \

Buffer Pool (Buffer Manager) Yönetimi [Linki](https://learn.microsoft.com/en-us/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object?view=sql-server-ver16) \

B+ Tree İndeks Yapısı ve Tasarımı [Linki](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide?view=sql-server-ver16) \
