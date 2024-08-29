# Lending1

Saya dapat memberikan contoh source code sederhana untuk aplikasi loan lending. Berikut ini adalah contoh dasar aplikasi loan lending yang dibangun dengan menggunakan **Python (Flask)** untuk backend dan **SQLite** sebagai database. Ini hanya contoh dasar dan tidak mencakup semua fitur yang akan Anda temukan di aplikasi pinjaman online komersial, tetapi bisa menjadi titik awal yang baik.

### 1. **Persiapan Lingkungan**
   Pastikan Anda memiliki Python dan pip terinstal di sistem Anda. Anda juga perlu menginstal Flask:

   ```bash
   pip install Flask
   ```

### 2. **Struktur Proyek**
   Buat direktori untuk proyek Anda dan strukturkan seperti ini:

   ```
   loan_lending/
   ├── app.py
   ├── database.db
   └── templates/
       └── index.html
   ```

### 3. **Backend (Flask + SQLite)**
   Buat file `app.py` dan tambahkan kode berikut:

   ```python
   from flask import Flask, request, jsonify, render_template
   from flask_sqlalchemy import SQLAlchemy

   app = Flask(__name__)
   app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
   db = SQLAlchemy(app)

   class User(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(80), nullable=False)
       email = db.Column(db.String(120), unique=True, nullable=False)

   class Loan(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
       amount = db.Column(db.Float, nullable=False)
       status = db.Column(db.String(20), default='pending')

   @app.route('/')
   def index():
       return render_template('index.html')

   @app.route('/api/users', methods=['POST'])
   def add_user():
       data = request.get_json()
       new_user = User(name=data['name'], email=data['email'])
       db.session.add(new_user)
       db.session.commit()
       return jsonify({'id': new_user.id, 'name': new_user.name, 'email': new_user.email}), 201

   @app.route('/api/loans', methods=['POST'])
   def apply_loan():
       data = request.get_json()
       new_loan = Loan(user_id=data['user_id'], amount=data['amount'])
       db.session.add(new_loan)
       db.session.commit()
       return jsonify({'id': new_loan.id, 'user_id': new_loan.user_id, 'amount': new_loan.amount, 'status': new_loan.status}), 201

   @app.route('/api/loans/<int:loan_id>', methods=['GET'])
   def get_loan(loan_id):
       loan = Loan.query.get_or_404(loan_id)
       return jsonify({'id': loan.id, 'user_id': loan.user_id, 'amount': loan.amount, 'status': loan.status})

   @app.route('/api/loans/<int:loan_id>', methods=['PUT'])
   def update_loan(loan_id):
       loan = Loan.query.get_or_404(loan_id)
       data = request.get_json()
       loan.status = data.get('status', loan.status)
       db.session.commit()
       return jsonify({'id': loan.id, 'user_id': loan.user_id, 'amount': loan.amount, 'status': loan.status})

   if __name__ == '__main__':
       db.create_all()
       app.run(debug=True)
   ```

### 4. **Frontend (HTML)**
   Di dalam folder `templates`, buat file `index.html` dengan kode berikut:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Loan Lending App</title>
       <style>
           body { font-family: Arial, sans-serif; }
           .container { max-width: 600px; margin: 50px auto; padding: 20px; border: 1px solid #ccc; }
           form { display: flex; flex-direction: column; }
           input, button { margin: 10px 0; padding: 10px; font-size: 16px; }
           button { cursor: pointer; }
       </style>
   </head>
   <body>
       <div class="container">
           <h2>Apply for a Loan</h2>
           <form id="loanForm">
               <input type="text" id="name" placeholder="Name" required>
               <input type="email" id="email" placeholder="Email" required>
               <input type="number" id="amount" placeholder="Loan Amount" required>
               <button type="submit">Submit</button>
           </form>
           <div id="response"></div>
       </div>
       <script>
           document.getElementById('loanForm').addEventListener('submit', function(event) {
               event.preventDefault();
               const name = document.getElementById('name').value;
               const email = document.getElementById('email').value;
               const amount = document.getElementById('amount').value;

               fetch('/api/users', {
                   method: 'POST',
                   headers: {
                       'Content-Type': 'application/json'
                   },
                   body: JSON.stringify({ name, email })
               })
               .then(response => response.json())
               .then(user => {
                   return fetch('/api/loans', {
                       method: 'POST',
                       headers: {
                           'Content-Type': 'application/json'
                       },
                       body: JSON.stringify({ user_id: user.id, amount })
                   });
               })
               .then(response => response.json())
               .then(loan => {
                   document.getElementById('response').innerText = `Loan application status: ${loan.status}`;
               })
               .catch(error => console.error('Error:', error));
           });
       </script>
   </body>
   </html>
   ```

### 5. **Menjalankan Aplikasi**
   Jalankan aplikasi dengan perintah berikut:

   ```bash
   python app.py
   ```

   Akses aplikasi di browser dengan membuka `http://localhost:5000`. Anda akan melihat halaman sederhana di mana Anda dapat mengajukan pinjaman. Data pinjaman akan disimpan di database SQLite yang berada di file `database.db`.

### 6. **Peningkatan dan Pengembangan Lanjutan**
   - **Validasi Input**: Tambahkan validasi input untuk memastikan bahwa data yang dimasukkan oleh pengguna valid.
   - **Keamanan**: Implementasikan fitur keamanan seperti autentikasi pengguna, enkripsi data, dan sanitasi input untuk mencegah serangan seperti SQL Injection.
   - **Dashboard Admin**: Buat antarmuka admin untuk memantau dan mengelola pinjaman.
   - **Integrasi API Pihak Ketiga**: Integrasikan dengan sistem pembayaran atau API cek kredit untuk memverifikasi kelayakan kredit pengguna.

Aplikasi ini adalah dasar sederhana untuk sistem loan lending. Anda bisa memperluasnya sesuai kebutuhan Anda. Jika Anda memerlukan bantuan tambahan, jangan ragu untuk bertanya!
