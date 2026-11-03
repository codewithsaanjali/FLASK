"""
My First Flask Application! 🚀
-------------------------------
I am learning Flask and built this single-file web application to practice:
- Basic Flask Routing & Dynamic Routes
- Inline HTML & Styling with render_template_string
- Handling Form Submissions (POST requests)
- SQLite Database integration with Flask-SQLAlchemy (CRUD operations)
"""

import os
from flask import Flask, render_template_string, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy

# 1. Initialize Flask App & Configure Database
app = Flask(__name__)
app.config['SECRET_KEY'] = 'my-beginner-secret-key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///my_notes.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# 2. Define Database Model (Note Taking Table)
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)

    def __repr__(self):
        return f'<Note {self.id}>'

# 3. HTML Template with Embedded CSS (All-in-one UI)
HTML_UI = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Flask App</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #eef2f5;
            display: flex;
            justify-content: center;
            padding-top: 50px;
        }
        .card {
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 450px;
        }
        h2 { color: #2c3e50; text-align: center; margin-bottom: 5px; }
        p.subtitle { text-align: center; color: #7f8c8d; font-size: 0.9em; margin-bottom: 25px; }
        form { display: flex; gap: 10px; margin-bottom: 20px; }
        input[type="text"] {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            outline: none;
        }
        button {
            padding: 10px 16px;
            border: none;
            background-color: #3498db;
            color: white;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover { background-color: #2980b9; }
        ul { list-style-type: none; padding: 0; }
        li {
            background: #f8f9fa;
            padding: 12px;
            border-radius: 6px;
            margin-bottom: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-left: 4px solid #3498db;
        }
        a.delete-link {
            color: #e74c3c;
            text-decoration: none;
            font-weight: bold;
            font-size: 0.9em;
        }
        a.delete-link:hover { text-decoration: underline; }
        .empty { text-align: center; color: #bdc3c7; font-style: italic; }
    </style>
</head>
<body>
    <div class="card">
        <h2>My Flask Notes 📝</h2>
        <p class="subtitle">Built while learning Python Flask</p>

        <!-- Form to Add Note -->
        <form action="{{ url_for('add_note') }}" method="POST">
            <input type="text" name="note_content" placeholder="Write a note..." required autocomplete="off">
            <button type="submit">Add</button>
        </form>

        <!-- Display List of Notes -->
        <ul>
            {% for note in notes %}
            <li>
                <span>{{ note.content }}</span>
                <a href="{{ url_for('delete_note', note_id=note.id) }}" class="delete-link">Delete</a>
            </li>
            {% else %}
            <p class="empty">No notes added yet!</p>
            {% endfor %}
        </ul>
    </div>
</body>
</html>
"""

# 4. Route Handlers

# Home Page Route (Read)
@app.route('/')
def home():
    notes = Note.query.all()
    return render_template_string(HTML_UI, notes=notes)

# Add Note Route (Create)
@app.route('/add', methods=['POST'])
def add_note():
    content = request.form.get('note_content')
    if content:
        new_note = Note(content=content)
        db.session.add(new_note)
        db.session.commit()
    return redirect(url_for('home'))

# Delete Note Route (Delete)
@app.route('/delete/<int:note_id>')
def delete_note(note_id):
    note = Note.query.get_or_404(note_id)
    db.session.delete(note)
    db.session.commit()
    return redirect(url_for('home'))

# 5. Run Application & Auto-Create Database Tables
if __name__ == '__main__':
    with app.app_context():
        db.create_all()  # Automatically builds database schema on startup
    print("Starting Flask Development Server...")
    app.run(debug=True)