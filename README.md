from flask import Flask, request, render_template, redirect, url_for, session
import sqlite3

app = Flask(__name__)
app.secret_key = "supersecretkey"

def init_db():
    conn = sqlite3.connect('vulnerable.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT, password TEXT)''')
    c.execute("INSERT OR IGNORE INTO users (username, password) VALUES ('admin', 'admin123')")
    conn.commit()
    conn.close()

init_db()

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
        conn = sqlite3.connect('vulnerable.db')
        c = conn.cursor()
        c.execute(query)
        user = c.fetchone()
        conn.close()
        if user:
            session['username'] = user[1]
            return redirect(url_for('dashboard'))
        else:
            return "Login failed. Invalid credentials."
    return render_template('login.html')

@app.route('/dashboard')
def dashboard():
    if 'username' in session:
        return f"Welcome, {session['username']}! <a href='/logout'>Logout</a>"
    else:
        return redirect(url_for('login'))

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('home'))

@app.route('/search', methods=['GET', 'POST'])
def search():
    if request.method == 'POST':
        query = request.form['query']
        return f"Search results for: {query}"
    return render_template('search.html')

@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    conn = sqlite3.connect('vulnerable.db')
    c = conn.cursor()
    c.execute(f"SELECT * FROM users WHERE id={user_id}")
    user = c.fetchone()
    conn.close()
    if user:
        return {"id": user[0], "username": user[1], "password": user[2]}
    else:
        return {"error": "User not found"}

if __name__ == '__main__':
    app.run(debug=True)
