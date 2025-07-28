# Keep Preferring Silent Waste — or Move to Smart Prediction? - Membangun Supervised Learning Untuk Memprediksi Tingkat Okupansi Ruangan

## Latar Belakang Proyek
Dalam perencanaan bangunan, terdapat berbagai aspek yang harus diperhatikan, terutama yang berkaitan dengan kenyamanan pengguna. Salah satu aspek kenyamanan yang penting adalah kenyamanan termal (thermal comfort), yang biasanya dicapai melalui penggunaan sistem HVAC (Heating, Ventilation, and Air Conditioning).

Namun, di balik kenyamanan tersebut, penggunaan sistem HVAC secara konvensional juga memberikan dampak negatif terhadap lingkungan. Permintaan energi listrik untuk menjalankan alat-alat penghawaan semakin meningkat setiap tahunnya, menjadikannya salah satu penyumbang terbesar dalam konsumsi energi bangunan.

Sebagai solusi, pendekatan berbasis permintaan (demand-driven) dapat menjadi alternatif untuk mengoptimalkan kinerja sistem HVAC. Dengan bantuan sensor lingkungan dan model machine learning, sistem dapat bekerja secara adaptif berdasarkan jumlah penghuni ruangan yang terdeteksi.

Dataset yang digunakan dalam proyek ini diperoleh dari [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/864/room+occupancy+estimation). Dataset ini merupakan hasil eksperimen selama 7 hari di sebuah ruangan berukuran 4,6 x 6 meter. Eksperimen dilakukan menggunakan sensor lingkungan non-intrusif yang terdiri dari lima jenis: suhu (temperature), kelembapan (humidity), kadar CO₂, intensitas cahaya (light), pergerakan (motion), dan suara (sound).

## Struktur Data dan Pemeriksaan Awal
 <img width="975" height="277" alt="image" src="https://github.com/user-attachments/assets/f830673f-26de-4c4e-99f9-ec059e3a2c47" />

Dataset terdiri dari 10.129 baris dan 19 kolom, yang merepresentasikan keluaran dari berbagai sensor lingkungan beserta lokasi penempatannya. Salah satu fitur, yaitu CO₂ slope, memiliki ketergantungan terhadap waktu, sehingga data ini dapat dikategorikan sebagai data deret waktu (time series).

Label (target) pada dataset adalah Room_Occupancy_Count, yaitu data ground truth yang diperoleh secara manual melalui pencatatan kehadiran. Label ini memiliki 4 nilai unik: 0, 1, 2, dan 3, yang merepresentasikan jumlah penghuni dalam ruangan.

## Metode

Langkah-langkah yang dilakukan dalam proyek ini meliputi:

### 1. Pembagian Data
- Mengeliminasi fitur `Date` dan `Time` karena tidak relevan dengan tujuan pemodelan.  
- Pembagian data dilakukan dengan proporsi 80% untuk data pelatihan dan 20% untuk data tes.  
- Pembagian data tidak dilakukan secara acak, melainkan mempertahankan urutan waktu untuk menghindari kebocoran data (data leakage).

### 2. Data Cleaning
- Tidak ditemukan nilai duplikat maupun *missing value*.  
- Nilai outlier ditemukan hampir di seluruh fitur, kecuali pada fitur suhu (temperature) dari lokasi S_1, S_3, dan S_4.  
- Outlier dianggap masih wajar secara kontekstual, sehingga dipertimbangkan untuk tidak ditangani secara khusus (*no handling applied*).

### 3. Exploratory Data Analysis (EDA)  
- Kehadiran fitur `S5_CO2_slope` menunjukkan bahwa data bersifat time series dan perlu memperhatikan urutan waktu saat pembagian data.  
- Terdapat sejumlah nilai outlier yang dinilai masih wajar sehingga tidak ditangani lebih lanjut.  
- Terdapat trade-off: fitur target pada data tes tidak mengandung nilai 1 untuk mempertahankan rentang waktu dan menghindari *data leakage*.  
- Distribusi pada data numerik tidak cukup representatif. Distribusi pada data train cenderung bervariasi dibandingkan data test sehingga berpotensi menyebabkan overfitting, sementara data kategori telah tersebar dengan baik.  
- Sensor suhu dan CO₂ meningkat seiring naiknya okupansi.  
- Sensor lampu memberikan informasi paling kuat untuk prediksi okupansi.

### 4. Pemodelan

Berdasarkan karakteristik data—seperti keberadaan outlier yang wajar, korelasi antar fitur, dan ketidakseimbangan pada target—dipilih empat algoritma utama sebagai kandidat model:
- Decision Tree (baseline)  
- SVM dengan kernel RBF  
- Random Forest  
- XGBoost  

Selanjutnya, data dikelompokkan menjadi dua pendekatan berdasarkan variasi kontribusi fitur:
- **Kelompok homogen**
- **Kelompok heterogen**

Setiap kelompok diuji melalui tiga eksperimen untuk mengevaluasi performa model:
- **Eksperimen pertama**: Melakukan resampling menggunakan teknik oversampling untuk menangani ketidakseimbangan kelas.  
- **Eksperimen kedua**: Pemodelan tanpa resampling, dengan menerapkan parameter `class_weight=balanced`.  
- **Eksperimen ketiga**: Hyperparameter tuning dilakukan pada dua model terbaik dari eksperimen sebelumnya.

### 5. Evaluasi Model

Evaluasi utama dilakukan menggunakan metode *cross-validation* untuk memastikan performa model yang stabil dan generalisabel.  

Model dari kelompok **homogen** menunjukkan akurasi yang tinggi, namun sangat bergantung pada fitur sensor lampu. Ketergantungan ini dinilai kurang representatif terhadap kondisi nyata yang lebih dinamis dan bervariasi.  

Sebaliknya, model dari kelompok **heterogen** dipilih sebagai solusi akhir karena mampu menunjukkan kerja sama antar fitur dan performa yang lebih stabil pada berbagai kondisi input.

Evaluasi akhir dilakukan menggunakan data pengujian, dengan catatan bahwa distribusi label pada data tersebut tidak sepenuhnya representatif. Hasil evaluasi divisualisasikan menggunakan **confusion matrix** dan **AUC-ROC** untuk memberikan gambaran menyeluruh terhadap kemampuan klasifikasi model.

### 6. Interpretability Model
- Metode interpretasi model menggunakan **Permutation Feature Importance** untuk mengetahui kontribusi tiap fitur terhadap prediksi.

## Ikhtisar Eksekutif
### Ikhtisar Temuan
 <img width="975" height="595" alt="image" src="https://github.com/user-attachments/assets/582aca3e-9b3c-481c-a739-bf69ed503c16" />

Model **XGBoost dengan optimasi hyperparameter** terbukti sebagai model terbaik, dengan rata-rata nilai **F1 Score (macro)** sebesar **88.3%** berdasarkan hasil *cross-validation*.  

Sensor **lampu** terbukti sangat membantu model dalam memprediksi tingkat okupansi, bahkan secara signifikan meningkatkan performa model. Namun setelah proses pelatihan, model justru menempatkan **sensor suara** sebagai fitur dengan kontribusi paling tinggi.

Bagi model XGBoost, keberadaan sensor suara berperan sebagai **pelengkap yang membuat model lebih robust dan adaptif** dalam konteks data dengan fitur yang lengkap.

## Rekomendasi

- **Prioritaskan penggunaan sensor lampu dan suara** untuk pemantauan okupansi, terutama saat sumber daya terbatas.  
- **Optimalkan kualitas dan kalibrasi sensor** agar model dapat bekerja secara maksimal dan konsisten.  
- **Jika metode berbasis waktu tetap digunakan**, disarankan untuk menambah jumlah data dan melakukan pemantauan secara *real-time* guna meningkatkan generalisasi, mengingat fitur seperti `CO2_slope` membutuhkan waktu pemrosesan sebelum dapat digunakan.  
- **Jika sensor CO₂ ditingkatkan menjadi lebih responsif**, maka pembagian data tidak perlu lagi mempertahankan urutan waktu. Dengan demikian, seluruh data dapat digunakan secara lebih fleksibel dan model dapat di-*deploy* secara baris per baris (*row-wise streaming*).

## Dampak Potensial

Model menghasilkan F1-score macro sebesar 88.3% berdasarkan cross-validation, dengan proporsi data okupansi (selain kelas 0) sebesar 18.77% dari total 10.129 data observasi selama 7 hari. Ini berarti terdapat sekitar 1.900 kejadian okupansi yang perlu dikenali oleh model.

Berdasarkan estimasi konservatif, deteksi okupansi yang benar mencapai ±1.678 kejadian. Jika setiap deteksi tersebut memungkinkan sistem HVAC untuk diaktifkan atau dinonaktifkan secara tepat waktu, maka sistem mampu menghindari penggunaan energi sebesar 1.2 kWh per kejadian.

Dengan tarif listrik komersial rata-rata Rp1.500/kWh, maka potensi penghematan energi yang dihasilkan adalah:

**± Rp3.020.400 dalam 7 hari operasional.**

Angka ini merupakan estimasi konservatif berdasarkan 1 jam penggunaan HVAC per kejadian. Nilai tersebut dapat meningkat jika model digunakan dalam sistem kontrol ruangan secara real-time dan berkelanjutan.
