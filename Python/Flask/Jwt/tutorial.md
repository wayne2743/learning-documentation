Flask-JWT (JSON Web Token) 是一個使用 Flask 框架建立 RESTful API 的工具，它提供了用戶認證和授權的功能。在本教學中，我們將使用 Flask-JWT 實現基於 JWT 的用戶認證功能。

# 安裝 Flask-JWT
使用 pip 安裝 Flask-JWT：

Copy code

```
pip install flask-jwt
```

## 創建 Flask 應用程序
創建一個名為 app.py 的 Python 文件並編寫以下代碼：


```
from flask import Flask
from flask_jwt import JWT

app = Flask(__name__)
app.config['SECRET_KEY'] = 'super-secret'

jwt = JWT(app)

```

在這裡，我們創建了一個 Flask 應用程序，設置了一個密鑰，並初始化了 Flask-JWT。

## 實現用戶認證
現在我們需要實現用戶認證。我們將使用一個假的用戶數據庫來模擬實際情況。我們將創建一個名為 users 的字典，其中包含兩個用戶名稱和密碼對。

```
users = {
    'user1': 'password1',
    'user2': 'password2'
}
```
接下來，我們需要創建一個函數，用於從用戶數據庫中檢索用戶名稱和密碼。如果找到用戶，則返回該用戶名稱對應的密碼，否則返回 None。

```
def authenticate(username, password):
    if username in users and password == users[username]:
        return username
```

接下來，我們需要創建一個函數，用於從 JWT 中解析出用戶名稱。這個函數將在每個請求中被調用，以檢查用戶是否已經通過認證。

```
def identity(payload):
    user_id = payload['identity']
    return {'user_id': user_id}
```

## 創建 RESTful API
現在我們已經準備好創建我們的 RESTful API。我們將創建兩個端點：/login 和 /protected。


## 登錄端點
/login 端點將接受用戶名稱和密碼，並使用 Flask-JWT 庫中的 jwt_required 裝飾器來創建一個 JWT。如果用戶名稱和密碼正確，則返回 JWT。


```
from flask import jsonify, request
from flask_jwt import jwt_required, current_identity

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username', None)
    password = request.json.get('password', None)
    if not username or not password:
        return jsonify({'message': 'Missing username or password'}), 400

    user_id = authenticate(username, password)

    if not user_id:
        return jsonify({'message': 'Invalid username or password'}), 401

    access_token = jwt.jwt_encode_callback({'identity': user_id})

    return jsonify({'access_token': access_token.decode('utf-8')})
```

在這裡，我們首先檢查請求中是否包含用戶名稱和密碼。如果沒有，則返回錯誤消息。否則，我們使用 authenticate 函數檢查用戶名稱和密碼是否正確。如果是，我們使用 Flask-JWT 中的 jwt_encode_callback 函數創建一個 JWT，並返回該 JWT。


## 受保護的端點
/protected 端點將使用 Flask-JWT 庫中的 jwt_required 裝飾器來保護。只有通過認證的用戶才能訪問該端點。在這個端點中，我們簡單地返回用戶名稱。

```
@app.route('/protected')
@jwt_required()
def protected():
    return jsonify({'user_id': current_identity['user_id']})
```
## 啟動 Flask 應用程序
現在我們已經創建了 Flask 應用程序和 RESTful API，我們可以使用以下代碼啟動 Flask 應用程序：

```
if __name__ == '__main__':
    app.run()
```
完整的 app.py 代碼如下：

```
from flask import Flask, jsonify, request
from flask_jwt import JWT, jwt_required, current_identity

app = Flask(__name__)
app.config['SECRET_KEY'] = 'super-secret'

users = {
    'user1': 'password1',
    'user2': 'password2'
}

def authenticate(username, password):
    if username in users and password == users[username]:
        return username

def identity(payload):
    user_id = payload['identity']
    return {'user_id': user_id}

jwt = JWT(app, authenticate, identity)

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username', None)
    password = request.json.get('password', None)

    if not username or not password:
        return jsonify({'message': 'Missing username or password'}), 400

    user_id = authenticate(username, password)

    if not user_id:
        return jsonify({'message': 'Invalid username or password'}), 401

    access_token = jwt.jwt_encode_callback({'identity': user_id})

    return jsonify({'access_token': access_token.decode('utf-8')})

@app.route('/protected')
@jwt_required()
def protected():
    return jsonify({'user_id': current_identity['user_id']})

if __name__ == '__main__':
    app.run()
```
現在，您可以使用以下命令啟動 Flask 應用程序：

Copy code
python app.py
恭喜！您已經學會了使用 Flask-JWT