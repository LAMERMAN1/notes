# notes

1. Установить Flask и Flask-SQLAlchemy:


pip install Flask
pip install Flask-SQLAlchemy


2. Создать файл с названием "app.py" и импортировать Flask и Flask-SQLAlchemy:

python
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy


3. Создать экземпляр Flask:

python
app = Flask(__name__)


4. Настроить базу данных:

python
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///notes.sqlite3'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)


5. Создать модель для заметок:

python
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    content = db.Column(db.Text)
    completed = db.Column(db.Boolean, default=False)


6. Создать маршруты для сохранения, удаления и изменения заметок:

python
@app.route('/')
def index():
    notes = Note.query.all()
    return render_template('index.html', notes=notes)

@app.route('/add', methods=['POST'])
def add_note():
    title = request.form['title']
    content = request.form['content']
    note = Note(title=title, content=content)
    db.session.add(note)
    db.session.commit()
    return redirect(url_for('index'))

@app.route('/delete/<int:id>')
def delete_note(id):
    note = Note.query.get_or_404(id)
    db.session.delete(note)
    db.session.commit()
    return redirect(url_for('index'))

@app.route('/update/<int:id>')
def update_note(id):
    note = Note.query.get_or_404(id)
    note.completed = not note.completed
    db.session.commit()
    return redirect(url_for('index'))


7. Создать шаблон для отображения заметок:

html
<!DOCTYPE html>
<html>
<head>
    <title>Notes</title>
</head>
<body>
    <h1>Notes</h1>
    <ul>
        {% for note in notes %}
        <li>
            {{ note.title }} - {{ note.content }} 
            <a href="{{ url_for('delete_note', id=note.id) }}">Delete</a>
            <a href="{{ url_for('update_note', id=note.id) }}">
                {% if note.completed %}
                Set Uncompleted
                {% else %}
                Set Completed
                {% endif %}
            </a>
        </li>
        {% endfor %}
    </ul>
    <form action="{{ url_for('add_note') }}" method="POST">
        <input type="text" name="title" placeholder="Title">
        <input type="text" name="content" placeholder="Content">
        <input type="submit" value="Add Note">
    </form>
</body>
</html>


8. Запустить приложение:

python
if __name__ == '__main__':
    app.run(debug=True)
