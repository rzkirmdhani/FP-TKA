# Final-Project-Cloud-Computing-Technology-2024
## Teknologi Komputasi Awan
### Kelas A-05
| No. | Nama | Nrp |
| :---         |     :---:      |          ---: |
| 1.   | Rizki Ramadhani     | 5027221013    |
| git diff     | git diff       | git diff      |
| git status   | git status     | git status    |
| git diff     | git diff       | git diff      |

# Permasalahan
Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: sentiment-analysis.py dengan spesifikasi sebagai berikut

## Endpoints:
1. Analyze Text

* Endpoint: POST /analyze
* Description: This endpoint accepts a text input and returns the sentiment score of the text.
* Request:
```
{
   "text": "Your text here"
}
```
* Response:
```
{
  "sentiment": <sentiment_score>
}
```
2. Retrieve History

* Endpoint: GET /history
* Description: This endpoint retrieves the history of previously analyzed texts along with their sentiment scores.
* Response:
```
{
 {
   "text": "Your previous text here",
   "sentiment": <sentiment_score>
 },
 ...
}
```
Kemudian juga disediakan sebuah Frontend sederhana menggunakan index.html dan styles.css dengan tampilan antarmuka sebagai berikut
<img width="1186" alt="image" src="https://github.com/rzkirmdhani/FP-TKA/assets/141987387/6f860aab-2b8e-4a23-acea-6c07e3e6137d">
Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah 1 juta rupiah per bulan (65 US$) konfigurasi cloud terbaik seperti apa yang bisa dibuat?
## Rancangan
Pada final project kami menggunakan Microsoft Azure.
![Screenshot 2024-06-20 235833](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/c33af9f9-a325-47bb-8f55-3947c815772e)

| **No.** | **Nama** | **Spesifikasi** | **Fungsi** | **Harga/Bulan** |
|-------------|-------------|-------------|-------------|-------------|
| Data 1-1    | Data 1-2    | Data 1-3    | Data 1-4    | Data 1-5    |
| Data 2-1    | Data 2-2    | Data 2-3    | Data 2-4    | Data 2-5    |
| Data 3-1    | Data 3-2    | Data 3-3    | Data 3-4    | Data 3-5    |
| Data 4-1    | Data 4-2    | Data 4-3    | Data 4-4    | Data 4-5    |
| Data 5-1    | Data 5-2    | Data 5-3    | Data 5-4    | Data 5-5    |

# Implementasi
## Konfigurasi VM1
![Screenshot 2024-06-20 122512](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/05a9c1e9-f56a-4b1e-9a4a-90b4cffdd4b0)

### (Backend)
1. Akses vm1 di PowerShell.
```
ssh -i ~/.ssh/id_rsa.pem azureuser@172.190.223.61
```
![Screenshot 2024-06-20 231930](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/4d1eb2e4-91fb-45aa-97cd-93bdb34e0f31)

Buat file sentiment-analysis.py
```
from flask import Flask, request, jsonify
from flask_pymongo import PyMongo
from textblob import TextBlob
from datetime import datetime

app = Flask(__name__)
app.config["MONGO_URI"] = "mongodb://localhost:27017/sentiment_analysis_db"
mongo = PyMongo(app)

@app.route('/analyze', methods=['POST'])
def analyze_sentiment():
    data = request.json
    text = data.get('text')
    if not text:
        return jsonify({'error': 'No text provided'}), 400

    analysis = TextBlob(text)
    sentiment = analysis.sentiment.polarity
    mongo.db.sentiments.insert_one({
        'text': text,
        'sentiment': sentiment,
        'date': datetime.utcnow()
    })
    return jsonify({'sentiment': sentiment}), 200

@app.route('/history', methods=['GET'])
def get_history():
    sentiments = list(mongo.db.sentiments.find().sort("date", -1))
    for sentiment in sentiments:
        sentiment['_id'] = str(sentiment['_id'])
    return jsonify(sentiments), 200

@app.route('/delete-history', methods=['POST'])
def delete_history():
    mongo.db.sentiments.delete_many({})
    return '', 204

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
### (MongoDB - Database)
1. Unduh MongoDB Community Edition.

![Screenshot 2024-06-20 234358](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/b4530542-3c35-4fde-a6dc-cd628396daf1)

2. Transfer file dari Download ke vm1.
```
scp -i ~/.ssh/id_rsa.pem C:/download/mongodb-linux-x86_64-7.0.11.tgz azureuser@172.190.223.61:~
```
3. Edit file konfigurasi /etc/mongod.conf
```
storage:
  dbPath: /var/lib/mongodb

systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

net:
  port: 27017
  bindIp: 127.0.0.1

processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```
4. Jalankan dan Verifikasi mongoDB
```
sudo systemctl start mongod
sudo systemctl status mongod
```
![Screenshot 2024-06-20 233853](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/6d2d213b-a4cc-4c20-9167-dab4aee92a01)

# Konfigurasi VM2
