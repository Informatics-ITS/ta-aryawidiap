# 🏁 Tugas Akhir (TA) - Final Project

**Nama Mahasiswa**: Arya Widia Putra \
**NRP**: 5025201016 \
**Judul TA**: Rancang Bangun Komponen Prediksi Risiko pada Aplikasi Mobile untuk Prediksi Risiko Serangan Jantung menggunakan Smartwatch Electrocardiogram dengan Pembelajaran Mesin \
**Dosen Pembimbing**: Prof. Ir. Ary Mazharuddin Shiddiqi, S.Kom., M.Comp.Sc., Ph.D., IPM. \
**Dosen Ko-pembimbing**: Dinh-Trung Vu (Asia University)

---

## 📺 Demo Aplikasi  

[![Demo Aplikasi](/img/HA-Prediction-5.jpg)](https://www.youtube.com/watch?v=f2EYBQR75K0)  
*Klik gambar di atas untuk menonton demo (demo dimulai pada 1:08)*

---


## Penjelasan Aplikasi  

> ### PERHATIAN ⚠️
>
> Aplikasi ini merupakan bagian dari riset penggunaan elektrokardiogram *smartwatch* untuk mendeteksi parameter kesehatan oleh Profesor Dinh-Trung Vu (Asia University) yang belum dipublish secara umum. Dengan demikian, kode sumber tidak dapat penulis sebar luaskan secara bebas.☹️
>
> Jika Anda tertarik untuk membuat aplikasi/riset sejenis, Anda dapat melihat penjelasan dan/atau menghubungi yang bersangkutan (kontak ko-pembimbing).😊

### Alat dan Bahan Pengembangan aplikasi

1. Macbook atau perangkat desktop sejenis dengan macOS, untuk mengembangkan aplikasi perangkat bergerak pada platform iOS.
2. iOS Simulator, dengan aplikasi Health dan mock data EKG bawaan.
3. Framework Flutter, untuk pengembangan aplikasi.
4. Android Studio, untuk menulis kode aplikasi perangkat bergerak.
5. Visual Studio Code, untuk menulis kode pengembangan server.
6. XAMPP, dengan Apache Server dan MySQL, untuk pengembangan basis data.
7. Framework Flask, dan library flask-sqlalchemy dan SQLalchemy sebagai framework pengembangan server.
8. Data dari Kaggle, [Anand et al. (2018)](https://www.kaggle.com/datasets/imnikhilanand/heart-attack-prediction) dan [Shayan Fazeli (2018)](https://www.kaggle.com/datasets/shayanfazeli/heartbeat), untuk bahan training model pembelajaran mesin untuk prediksi anomali dan serangan jantung.

### Penjelasan Fungsi Prediksi Risiko Serangan Jantung

Fungsi prediksi risiko serangan jantung dilakukan di halaman formulir prediksi risiko serangan jantung. Halaman ini menerima deteksi anomali yang berjalan saat tombol pada halaman Home ditekan. Deteksi anomali ini dilakukan dengan model tflite yang terbungkus pada aplikasi *mobile*. Kurang lebih, proses prediksi adalah sebagai berikut.

```plain
Pengguna memilih rentang waktu EKG -> variabel penyimpan EKG terisi oleh data EKG -> tombol 'Predict Heart Attack' ditekan -> variabel EKG diteruskan ke model deteksi anomali (CNN autoencoders) -> hasil prediksi dikirim ke halaman risiko serangan jantung
```

Pada halaman formulir, hasil ini direpresentasikan sebagai 'Anomalous ECG', yang dapat bernilai 'Not detected' or 'Detected'. Lalu, dalam formulir ini, pengguna diminta untuk mengisi data kesehatan pribadi, yaitu jenis kelamin (‘Sex’), umur (‘Age’), kebiasaan merokok (‘Do you smoke?’), dan adanya nyeri dada (‘Do you have any chest pain?’).

![alt text](/img/HA-Prediction-5.jpg)
Gambar 1. Halaman formulir prediksi risiko serangan jantung.

Komponen utama dari formulir prediksi risiko serangan jantung adalah *widget* Form. Form digunakan untuk membuat formulir yang dapat diisi dengan berbagai *widget* lain. Salah satu fungsi utama Form adalah memvalidasi nilai-nilai *input*.

```dart
@override
Widget build(BuildContext context) {
  ...
  return Form(
    key: _formKey,
    child: Column(
      children: [
        // your input fields here...
        Column(
          children: [
            ...
            ElevatedButton(
              onPressed: () {
                if (_formKey.currentState!.validate()) {
                  _age = double.parse(_ageController.text);
                  // http get with the datas
                  final heartAttackData = HeartAttackData(...);
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => HeartAttackPredictionResult(
                        heartAttackData: heartAttackData,
                      ),
                    ),
                  );
                }
              },
              child: const Text('Get Prediction'),
            ),
          ],
        ),
      ],
    ),
  );
}
```

Kode Sumber 1. Potongan kode untuk formulir prediksi risiko serangan jantung (heart_attack_prediction_form.dart)

Saat pengguna menekan tombol ‘Get Prediction’, data dikirim ke server dan diproses menggunakan model pembelajaran mesin (*decision forest*, *exported* ke format Pickle) untuk prediksi risiko serangan jantung. Jika prediksi mengembalikan nilai 0 dari server, aplikasi akan menganggapnya sebagai false dan menampilkan layar tidak ada risiko (‘No risk’), seperti Gambar 2(a). Jika server mengembalikan nilai 1 sebagai hasil prediksi, aplikasi akan menganggapnya sebagai true dan menampilkan layar ada risiko (‘At risk’), seperti Gambar 2(b).

| ![Halaman hasil prediksi ketika pengguna diprediksi berisiko](/img/HA-Prediction-4.jpg)    | ![alt text](/img/HA-Prediction-3.jpg) |
| -------- | ------- |
| (a)  | (b)    |

Gambar 2. Halaman hasil prediksi risiko serangan jantung. (a) menunjukkan halaman jika model pembelajaran mesin memprediksi tidak ada risik serangan jantung, sedangkan (b) menunjukkan jika ada.

Dalam kode, halaman hasil prediksi risiko serangan jantung disokong dengan beberapa widget. Data dikirim ke server, lalu server mengembalikan hasil prediksi ke aplikasi. Dengan demikian, proses ini adalah proses *asynchronous*. Untuk menangani hal tersebut, penulis menggunakan widget ChangeNotifierProvider, FutureBuilder, dan Provider (HeartAttackProvider). ChangeNotifierProvider akan memperbarui nilai state dan widget yang menggunakan state tersebut saat state tersebut berubah, yang akan terjadi saat hasil proses prediksi dikembalikan dari server (Kode Sumber 2) . State ini dimiliki dan dikelola oleh HeartAttackProvider.

```dart
ChangeNotifierProvider.value(
  value: HeartAttackProvider(),
  child: HeartAttackResultPoster(
    heartAttackData: widget.heartAttackData
  )
)
```

Kode Sumber 2. Penggunaan ChangeNotifierProvider untuk menangani data *asynchronous* yang akan dikirim ke *widget* HeartAttackResultPoster.

FutureBuilder berfungsi untuk memanggil fungsi *asynchronous* getHeartAttackPrediction milik HeartAttackProvider (Kode Sumber 3). Parameter builder FutureBuilder akan membangun widget yang akan ditampilkan pada antarmuka pengguna. Dalam hal ini, ketika data masih diproses (snapshot.connectionState == ConnectionState.waiting), tampilan antarmuka akan menampilkan loading icon. Jika tidak, tampilan antarmuka akan diatur oleh widget Consumer (Kode Sumber 5).

```dart
FutureBuilder(
  future: Provider.of<HeartAttackProvider>(context, listen: false)
      .getHeartAttackPrediction(_heartAttackData),
  builder: (context, snapshot) => snapshot.connectionState ==
          ConnectionState.waiting
      ? const Center(
          child: CircularProgressIndicator(),
        )
      : Consumer<HeartAttackProvider>(
         ...
        )
)
```

Kode Sumber 3. *Widget* FutureBuilder untuk menangani koneksi dan hasil dari fungsi *asynchronous* *provider* HeartAttackProvider.

Fungsi getHeartAttackPrediction akan mengirim data dari formulir prediksi risiko dalam bentuk JSON ke server (Kode Sumber 4). Fungsi ini akan menunggu response dari server, yaitu hasil prediksi. Setelahnya, fungsi ini akan mengubah state _heartAttackDataPrediction, yang mana akan digunakan di widget lainnya.

```dart
Future<void> getHeartAttackPrediction(HeartAttackData data) async {
  try {
    Map<String, dynamic> request = {
      "sex": data.sex,
      "age": data.age,
      "chest_pain": data.chest_pain,
      "smoking": data.smoking,
      "anomaly": data.anomaly,
    };
    final headers = {'Content-Type': 'application/json'};
    final response = await http.post(Uri.parse(url),
        headers: headers, body: json.encode(request));
    Map<String, dynamic> responsePayload = json.decode(response.body);
    final heartAttackDataPrediction = responsePayload["prediction"];
    _heartAttackDataPrediction = [
      heartAttackDataPrediction.toString().toLowerCase() != "false"
    ];
    debugPrint(_heartAttackDataPrediction[0].toString());
  } on Exception catch (e) {
    print(e);
  }
  notifyListeners();
}
```

Kode Sumber 4. Fungsi *asynchronous* getHeartAttackPrediction pada HeartAttackProvider untuk mendapatkan hasil prediksi dari server.

Widget Consumer akan dibuat saat server sudah mengembalikan response dan data diolah oleh fungsi getHeartAttackPrediction. Jika hasil kosong (heartAttackProvider.heartAttackDataPrediction.isEmpty), antarmuka akan menampilkan ‘Prediction failed.’. Jika hasil yang dikembalikan adalah true atau false, nilai hasil akan diteruskan ke widget HeartAttachHero yang akan menghasilkan antarmuka seperti pada Gambar 2.

```dart
Consumer<HeartAttackProvider>(
  child: Center(
    heightFactor: MediaQuery.of(context).size.height * 0.03,
    child: Text(
      'Prediction failed.',
      style: inputTextTextStyle,
    ),
  ),
  builder: (context, heartAttackProvider, child) =>
      heartAttackProvider.heartAttackDataPrediction.isEmpty
          ? child as Widget
          : Padding(
              padding: const EdgeInsets.only(top: 20),
              child: Container(child: () {
                if (heartAttackProvider
                        .heartAttackDataPrediction[0] ==
                    true) {
                  return HeartAttackHero(result: true,);
                }
                return HeartAttackHero(result: false,);
              }()),
            ),
)
```

Kode Sumber 5. *Widget* Consumer yang menangani hasil dari fungsi asinkronus pada Kode Sumber 4.

---

## ⁉️ Pertanyaan?

Hubungi:
- Penulis: aryawidiaputra2002[at]gmail.com
- Pembimbing Utama: ary.shiddiqi[at]if.its.ac.id
- Ko-pembimbing: tonyvu[at]asia.edu.tw
