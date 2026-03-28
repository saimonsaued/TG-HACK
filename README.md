from flask import Flask, session, redirect, request, jsonify, render_template_string
import random
import time
import threading

app = Flask(__name__)
app.secret_key = "1212"

PASSWORD = "1212"

current_signal = {
    "number": 0,
    "size": "",
    "color": ""
}

last_update = 0

def generate_signal():
    global current_signal, last_update
    while True:
        number = random.randint(0, 9)

        size = "BIG" if number >= 5 else "SMALL"

        if number % 2 == 0:
            color = "RED"
        else:
            color = "GREEN"

        current_signal = {
            "number": number,
            "size": size,
            "color": color
        }

        last_update = int(time.time())
        time.sleep(30)

threading.Thread(target=generate_signal, daemon=True).start()

# 🔐 Stylish Login Page
@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        if request.form.get("password") == PASSWORD:
            session["login"] = True
            return redirect("/signal")

    return '''
    <html>
    <head>
    <title>Login</title>
    <style>
        body {
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: Arial;
            color: white;
        }
        .box {
            background: rgba(255,255,255,0.1);
            padding: 40px;
            border-radius: 15px;
            text-align: center;
            backdrop-filter: blur(10px);
        }
        input {
            padding: 10px;
            border: none;
            border-radius: 5px;
            margin-top: 10px;
        }
        button {
            margin-top: 15px;
            padding: 10px 20px;
            background: #00c6ff;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
        }
    </style>
    </head>
    <body>
        <div class="box">
            <h2>🔐 PRO SIGNAL LOGIN</h2>
            <form method="post">
                <input type="password" name="password" placeholder="Enter Password"/><br>
                <button>Login</button>
            </form>
        </div>
    </body>
    </html>
    '''

# 📊 Stylish Signal Page
@app.route("/signal")
def signal():
    if not session.get("login"):
        return redirect("/")

    return render_template_string("""
    <html>
    <head>
    <title>Signal</title>
    <style>
        body {
            background: #0f172a;
            color: white;
            text-align: center;
            font-family: Arial;
        }
        .card {
            margin-top: 50px;
            background: #1e293b;
            padding: 30px;
            border-radius: 15px;
            display: inline-block;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        .number {
            font-size: 60px;
            font-weight: bold;
        }
        .big { color: #00ffcc; }
        .small { color: #ffcc00; }
        .red { color: #ff4d4d; }
        .green { color: #00ff99; }
    </style>
    </head>
    <body>

    <h1>🔥 PRO SIGNAL 🔥</h1>

    <div class="card">
        <div class="number" id="num">0</div>
        <div id="size"></div>
        <div id="color"></div>
        <div id="timer"></div>
    </div>

    <script>
    function loadSignal(){
        fetch('/data')
        .then(res => res.json())
        .then(d => {
            document.getElementById('num').innerText = d.number;

            document.getElementById('size').innerHTML =
                "<span class='"+d.size.toLowerCase()+"'>"+d.size+"</span>";

            document.getElementById('color').innerHTML =
                "<span class='"+d.color.toLowerCase()+"'>"+d.color+"</span>";

            let remaining = Math.floor(30 - (Date.now()/1000 - d.time));
            document.getElementById('timer').innerText =
                "Next Signal in: " + remaining + "s";
        });
    }

    setInterval(loadSignal, 1000);
    </script>

    </body>
    </html>
    """)

@app.route("/data")
def data():
    return jsonify({
        "number": current_signal["number"],
        "size": current_signal["size"],
        "color": current_signal["color"],
        "time": last_update
    })

app.run(host="0.0.0.0", port=5000)
