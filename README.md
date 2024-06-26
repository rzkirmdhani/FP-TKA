# Final-Project-Cloud-Computing-Technology-2024
## Teknologi Komputasi Awan
### Kelas A-05
| No. | Nama | Nrp |
| :--- |     :---:      |          ---: |
| 1.   | Rizki Ramadhani     | 5027221013    |
| 2.   | Angella Christie    | 5027221047    |
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

### Tabel Harga
Total kami menggunakan 2 VM. VM1 sebagai Backend dan VM2 sebagai Frontend. Berikut untuk spesifikasi VM yang kami gunakan.
| **No.** | **Nama** | **Spesifikasi** | **Fungsi** | **Harga/Bulan** |
|---------|-------------|-------------|-------------|-------------|
| 1. | VM1 | Size Standard B1s, 1vCPUs, Ram 1GB| Backend    | $7,59|
| 2. | VM2 | Size Standard B1s, 1vCPUs, Ram 1GB| Frontend    | $7,59|
|  |  | Total|     | $15,18|

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
5. Jalankan sentiment-analysis.py
```
python3 sentiment-analysis.py
```
![Screenshot 2024-06-21 135217](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/826828c3-4726-4c49-bf68-05ed8090a11c)

## Konfigurasi VM2
![Screenshot 2024-06-20 122652](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/00a55d89-fe3b-4633-9657-342e4bd8ef82)
### Nginx
1. Akses VM2 di Powershell.
```
ssh -i ~/.ssh/id_rsa.pem azureuser@172.190.221.74
```
2. Edit file nginx pada /etc/nginx/sites-available/default
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    location /analyze {
        proxy_pass http://172.190.223.61:5000/analyze;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /history {
        proxy_pass http://172.190.223.61:5000/history;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /delete-history {
        proxy_pass http://172.190.223.61:5000/delete-history;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
3. Restart nginx
```
sudo systemctl restart nginx
```
4. Ubah file index.html pada /var/www/html/index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sentiment Analysis</title>
    <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1 class="text-center">Sentiment Analysis</h1>
        <div class="row">
            <div class="col-md-6">
                <div class="form-container">
                    <form id="sentiment-form">
                        <div class="form-group">
                            <textarea id="text-input" class="form-control" rows="4" placeholder="Enter text here..."></textarea>
                        </div>
                        <button type="submit" class="btn btn-success btn-block">Analyze</button>
                    </form>
                    <p id="result" class="text-center mt-4"></p>
                </div>
            </div>
            <div class="col-md-6">
                <div class="history-container">
                    <h2 class="text-center">History</h2>
                    <ul id="history" class="list-group mt-3"></ul>
                    <button id="clear-history" class="btn btn-danger btn-block mt-3">Clear History</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.getElementById('sentiment-form').addEventListener('submit', async function(event) {
            event.preventDefault();
            const text = document.getElementById('text-input').value;
            try {
                const response = await fetch('/analyze', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ text }),
                });
                if (!response.ok) throw new Error('Network response was not ok');
                const result = await response.json();
                const resultElement = document.getElementById('result');
                resultElement.textContent = `Sentiment Score: ${result.sentiment}`;
                resultElement.className = result.sentiment === 0 ? 'bg-danger text-white' : 'bg-success text-white';
                fetchHistory();
            } catch (error) {
                console.error('There has been a problem with your fetch operation:', error);
            }
        });

        async function fetchHistory() {
            try {
                const response = await fetch('/history');
                if (!response.ok) throw new Error('Network response was not ok');
                const history = await response.json();
                const historyList = document.getElementById('history');
                historyList.innerHTML = '';
                history.forEach(item => {
                    const listItem = document.createElement('li');
                    listItem.className = `list-group-item history-item ${item.sentiment === 0 ? 'bg-danger text-white' : 'bg-success text-white'}`;
                    listItem.textContent = `Text: ${item.text}, Sentiment: ${item.sentiment}`;
                    historyList.appendChild(listItem);
                });
            } catch (error) {
                console.error('There has been a problem with your fetch operation:', error);
            }
        }

        document.getElementById('clear-history').addEventListener('click', async function() {
            try {
                const response = await fetch('/delete-history', { method: 'POST' });
                if (response.ok) {
                    fetchHistory();
                }
            } catch (error) {
                console.error('Error clearing history:', error);
            }
        });

        // Fetch history on page load
        fetchHistory();
    </script>
</body>
</html>
```

## Load Balancer
Pada final project saat ini kami menggunakan Load Balancer milik Azure.
![Screenshot 2024-06-20 122731](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/3a286103-0d08-499e-92d5-c48dbed5c773)
1. Buat Load Balancer yang dimiliki oleh Azure
![Screenshot 2024-06-21 141413](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/ec790cdd-0b9d-4a0b-8131-adf3c2530c51)
2. Buat PublicIP Load Balancer
![Screenshot 2024-06-20 122759](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/e52c5c17-f608-4591-8fa7-81479871c819)
3. Tambahkan Backend pools yaitu VM1 dan VM2
![Screenshot 2024-06-20 122821](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/859fd4b2-8dd1-4fbc-8f9d-be747d00f844)
4. Buat Load Balncing Rules
![image](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/c13adf56-94b3-4cbc-9035-6eab26dd650a)
![image](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/7c094c83-7768-451f-8465-5f21913ec4db)

* Jadi VM1 akan menjalankan Mongodb dan App Flask, mengolah data Sentiment.
* VM2 menjalankan Nginx dan menampilkan frontend, mengarahkan permintaan ke aplikasi Flask di VM1.
* Kunjungi PublicIP vm2 yaitu frontend atau juga bisa pada PublicIP Load balancer dan test lakuakan analisis.
* Frontend VM2 atau Load Balancer.
```
172.190.221.74   X   20.185.234.5
```

![Screenshot 2024-06-21 142940](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/19c1516f-718c-4563-a7b0-a954c4488ed4)
* Pada History
```
http://172.190.221.74/history
```
![image](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/4d1c1811-0ca3-4b8f-a4cf-912751c97b2a)

#  Pengujian endpoint API
Menggunakan Postman untuk Pengujian endpoint API.
1. Pada Analyze
```
http://172.190.221.74/analyze
```
![Screenshot 2024-06-21 150901](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/0a6ed826-8f7a-4019-9b59-b9d6caab2979)
2. Pada History
```
http://172.190.221.74/history
```
![Screenshot 2024-06-21 151122](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/773ef56f-52e5-4122-bded-1245fc3cf687)



# Testing 
1. Locust testing dengan waktu 60 detik, maximum RPS 31.3, failure 0%
   
   ![Gambar WhatsApp 2024-06-20 pukul 17 48 14_1fc8f9fa](https://github.com/rzkirmdhani/FP-TKA/assets/131789727/45ce0230-dd0e-4972-b17d-3d13f2557f9c)

2. Locust testing dengan spawn rate 50, waktu 60 detik, maximum peak concurrency 100, maximum RPS 34.3, failure 0%.

   ![Gambar WhatsApp 2024-06-20 pukul 17 48 14_9bbade84](https://github.com/rzkirmdhani/FP-TKA/assets/131789727/e20e5943-f913-4207-bd1e-2650ae810b40)

3. Locust testing dengan spawn rate 100, waktu 60 detik, maximum peak concurrency 150, maximum RPS 41.5, failure 0%.

   ![Gambar WhatsApp 2024-06-20 pukul 21 40 27_e7e4dc27](https://github.com/rzkirmdhani/FP-TKA/assets/131789727/6e0f5900-64cc-45fc-82d0-50fe4764be84)

4. Locust testing dengan spawn rate 200, waktu 60 detik, maximum peak concurrency 250, maximum RPS 54.25, failure 0%.

   ![Gambar WhatsApp 2024-06-20 pukul 17 56 11_72249c94](https://github.com/rzkirmdhani/FP-TKA/assets/131789727/6873daad-0206-4810-b0ee-382b206949bb)

5. Locust testing dengan spawn rate 500, waktu 60 detik, maximum peak concurency 505, maximum RPS 159.75, failure 24.5%.
![500](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/d1b61fe8-69f2-4d81-a0fb-ca1421f39de1)

   
# Kesimpulan dan Saran
Performansi sistem meningkat seiring dengan peningkatan spawn rate dan concurrency, dengan detail sebagai berikut:
   - Pada spawn rate 50, maximum peak concurrency mencapai 100 dan maximum RPS sebesar 34.3.
   - Pada spawn rate 100, maximum peak concurrency mencapai 150 dan maximum RPS sebesar 41.5.
   - Pada spawn rate 200, maximum peak concurrency mencapai 250 dan maximum RPS sebesar 54.25.
   - Pada spawn rate 500, maximum peak concurrency mencapai 500 dan maximum RPS sebesar 120.5.

# Revisi
Melakukan Rezise spesifikasi yang lebih tinggi dari Total $15,18 menjadi $60.74 dengan spesifikasi sebagai berikut.
| **No.** | **Nama** | **Spesifikasi** | **Fungsi** | **Harga/Bulan** |
|---------|-------------|-------------|-------------|-------------|
| 1. | VM1 | Size Standard B2s, 2vCPUs, Ram 4GB| Backend    | $30,37|
| 2. | VM2 | Size Standard B2s, 2vCPUs, Ram 4GB| Frontend    | $30,37|
|  |  | Total|     | $$60.74|

Dengan melakukan Rezise VM berikut hasil Locust testing.
1. Locust testing dengan waktu 60 detik, maximum RPS 89.5, failure 0%
   ![Screenshot 2024-06-26 165248](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/d0f1a0a1-70c8-4184-b776-a19c09214454)
2. Locust testing dengan spawn rate 50, waktu 60 detik, maximum peak concurrency 700, maximum RPS 131.7, failure 0%.
   ![Screenshot 2024-06-26 165459](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/f99219af-6607-4995-b33a-c4cc0527593b)
3. Locust testing dengan spawn rate 100, waktu 60 detik, maximum peak concurrency 650, maximum RPS 170, failure 0%.
   ![Screenshot 2024-06-26 165608](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/c7415784-48e8-40d9-92b8-298a54f28bd9)
4. Locust testing dengan spawn rate 200, waktu 60 detik, maximum peak concurrency 600, maximum RPS 224, failure 0%.
   ![Screenshot 2024-06-26 012820](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/878e2e05-b541-4758-89b7-064611812035)
5. Locust testing dengan spawn rate 500, waktu 60 detik, maximum peak concurency 650, maximum RPS 226.4, failure 0%.
   ![Screenshot 2024-06-26 013108](https://github.com/rzkirmdhani/FP-TKA/assets/141987387/9c87243c-b81e-4e78-a344-6f1951413af3)

# Kesimpulan dan Saran
Gunakan spesifikasi VM dengan maksimal asalkan tidak melebihi budget yang di tentukan.
