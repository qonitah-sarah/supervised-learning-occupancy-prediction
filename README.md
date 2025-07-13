
# Keep Preferring Silent Waste — or Move to Smart Prediction? - Membangun Supervised Learning Untuk Memprediksi Tingkat Okupansi Ruangan

## Deskripsi  
Proyek ini bertujuan membangun model supervised learning untuk memprediksi tingkat okupansi ruangan dengan pendekatan non-instrusive enviromental sensors yang terdiri dari lima jenis sensor: temperatur, gerak, kelembapan, CO2, suara, dan lampu. Kombinasi antara sensor dan model adalah sebagai upaya untuk menciptakan sistem HVAC yang bekerja berdasarkan permintaan (demand-driven) sehingga kenyamanan termal dan efisiensi energi dapat berjalan beriringan.

## Dataset  
Data diperoleh dari [UCI Occupancy Detection Dataset](https://archive.ics.uci.edu/dataset/864/room+occupancy+estimation) yang merupakan hasil dari eksperimen berisi rekaman sejumlah sensor selama 10 hari. 

## Exploratory Data Analysis (EDA)  
- Data memiliki kolom S5_co2_slope, sehingga data memiliki sifat time series dan perlu memperhatikan urutan waktu saat pembagian data.
- Terdapat sejumlah nilai outlier yang merupakan nilai yang wajar sehingga dipertimbangkan untuk tidak dihandle.
- Trade off yang harus diterima: Fitur target pada data test tidak memiliki nilai 1 untuk mempertahankan rentang waktu dan menghindari data leakage.
- Distribusi pada data numerik tidak cukup representatif. Ditribusi pada data train cenderung bervariasi dibandingkan data test sehingga berpotensi overfitting, sementara data kategori telah tersebar dengan baik.
- Sensor suhu dan CO2 meningkat seiring naiknya okupansi.  
- Sensor lampu memberikan informasi paling kuat untuk prediksi okupansi. 

## Model dan Hasil  
- Melakukan pengujian di 2 kelompok fitur: homogen dan heterogen. Homogen yang diuji adalah sebanyak 7 kelompok dan heterogen berjumlah 4 kombinasi fitur yang diuji secara bertahap. 
- 2 kelompok fitur diuji menggunakan 4 model: Decision Tree (baseline), SVM RBF. Random Forest, dan XGBoost. SVM RBF merupakan model satu-satunya yang dilakukan scaling dengan Robust Scaler karena model yang lainnya robust terhadap outlier.
- Karena data test tidak memiliki nilai yang lengkap, maka metode evaluasi utama yang digunakan adalah cross validation dengan metrik F1 score macro.
- Model XGBoost digunakan dengan F1 score terbaik.  
- Sensor suara dan lampu adalah yang paling penting dalam prediksi.

## Rekomendasi  
- Prioritaskan penggunaan sensor lampu dan suara untuk monitoring okupansi saat sumber daya terbatas. 
- Mengoptimalkan kualitas sensor agar model dapat bekerja optimal.
- Jika alat sensor tetap digunakan, maka metode sama dengan yang dilakukan sebelumnya yaitu berbasis waktu namun perlu dilakukan penambahan data dan dipantau secara real time agar tergenalisir dengan baik dan menghadapi resiko jika tidak dapat dideploy secara baris per baris karena perlu diproses beberapa waktu untuk menghasilkan co2 slope.
- Jika alat sensor co2 diupgrade menjadi lebih responsif, maka pembagian data tidak perlu memperhatikan rentang waktu, sehingga data yang ada dapat digunakan secara efektif dan model dapat di-deploy secara baris per baris.


