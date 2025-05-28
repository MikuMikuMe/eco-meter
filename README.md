# eco-meter

To create an eco-meter web application in Python, we can use the Flask framework for building the web application and integrate it with a database like SQLite for data storage and manipulation. Here's a complete Python program outlining a basic version of the eco-meter application:

### Project Structure
- `app.py`: Main application file.
- `templates/`: Directory for HTML templates.
- `static/`: Directory for static files like CSS and JavaScript.
- `database.db`: SQLite database file.

### Program Code

#### app.py
```python
from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3
import os

app = Flask(__name__)
app.secret_key = 'superSecretKey'
DATABASE = 'database.db'

# Ensure database and table exist
def init_db():
    with sqlite3.connect(DATABASE) as conn:
        cur = conn.cursor()
        cur.execute('''CREATE TABLE IF NOT EXISTS users (
                            id INTEGER PRIMARY KEY AUTOINCREMENT,
                            name TEXT NOT NULL,
                            email TEXT UNIQUE NOT NULL,
                            footprint INTEGER DEFAULT 0
                       )''')
        cur.execute('''CREATE TABLE IF NOT EXISTS recommendations (
                            id INTEGER PRIMARY KEY AUTOINCREMENT,
                            recommendation TEXT NOT NULL
                       )''')

# Retrieve the database connection
def get_db():
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row
    return conn

@app.route('/')
def index():
    # Show personalized recommendations and current footprint
    user = None
    recommendations = []
    try:
        db = get_db()
        user_data = db.execute('SELECT * FROM users WHERE id = ?', (1, )).fetchone()
        if user_data:
            user = dict(user_data)
        recommendations_data = db.execute('SELECT * FROM recommendations').fetchall()
        recommendations = [dict(row) for row in recommendations_data]
    except Exception as e:
        flash(f"An error occurred: {e}")
    finally:
        db.close()

    return render_template('index.html', user=user, recommendations=recommendations)

@app.route('/update', methods=['POST'])
def update():
    # Update carbon footprint and provide analytic response
    if request.method == 'POST':
        try:
            footprint = request.form.get('footprint', type=int)
            db = get_db()
            db.execute('UPDATE users SET footprint = ? WHERE id = ?', (footprint, 1))
            db.commit()
            flash("Carbon footprint updated successfully!")
        except Exception as e:
            flash(f"An error occurred while updating: {e}")
        finally:
            db.close()

    return redirect(url_for('index'))

@app.route('/profile', methods=['POST'])
def profile():
    # Create or update user profile
    if request.method == 'POST':
        try:
            name = request.form['name']
            email = request.form['email']
            db = get_db()
            db.execute('INSERT OR IGNORE INTO users (name, email) VALUES (?, ?)', (name, email))
            db.commit()
            flash("Profile updated successfully!")
        except Exception as e:
            flash(f"Profile update failed: {e}")
        finally:
            db.close()

    return redirect(url_for('index'))

if __name__ == '__main__':
    if not os.path.exists(DATABASE):
        init_db()
    app.run(debug=True)
```

#### Templates and Static Files

**templates/index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Eco-Meter</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <h1>Welcome to Eco-Meter</h1>
    <form action="{{ url_for('profile') }}" method="post">
        <h3>Create/Update Profile</h3>
        <input type="text" name="name" placeholder="Name" required>
        <input type="email" name="email" placeholder="Email" required>
        <button type="submit">Submit</button>
    </form>

    {% if user %}
    <h2>Hello, {{ user.name }}!</h2>
    <p>Your current carbon footprint is: {{ user.footprint }} kg CO2</p>
    <form action="{{ url_for('update') }}" method="post">
        <input type="number" name="footprint" placeholder="New Footprint Value" required>
        <button type="submit">Update Footprint</button>
    </form>
    {% endif %}

    <h3>Personalized Recommendations</h3>
    <ul>
    {% for rec in recommendations %}
        <li>{{ rec.recommendation }}</li>
    {% endfor %}
    </ul>
    
    {% with messages = get_flashed_messages() %}
      {% if messages %}
        <ul class="flashes">
          {% for message in messages %}
            <li>{{ message }}</li>
          {% endfor %}
        </ul>
      {% endif %}
    {% endwith %}
</body>
</html>
```

**static/style.css**
```css
body {
    font-family: Arial, sans-serif;
}

h1, h2, h3 {
    color: #2c3e50;
}

form {
    margin: 10px 0;
}

input, button {
    padding: 8px;
    margin: 5px 0;
}

ul.flashes {
    color: red;
}
```

### Notes
- **Database Persistence**: Ensure the database file `database.db` is persisted, which may require different handling if deployed.
- **Error Handling**: The program includes basic error handling with flash messages for user feedback.
- **Security**: The example application uses a hardcoded secret key; for production, you should use a securely generated secret key and store it in a safe place like environment variables.

Please ensure that you have Flask installed in your Python environment. You can install it via pip:

```bash
pip install flask
```

This is a basic setup to start with. Depending on requirements, this can be expanded to include detailed analytics, user authentication, and more complex recommendation logic.