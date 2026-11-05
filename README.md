# a-apoo-modelagem-
cat << 'EOF' > setup.sh
#!/bin/bash

# Interrompe em caso de erro
set -e

REPO_NAME="sistema-bmvc-estoque"

echo "==> 1. Criando diretório do projeto..."
mkdir -p "$REPO_NAME"
cd "$REPO_NAME"

echo "==> 2. Inicializando repositório Git..."
git init

echo "==> 3. Criando o arquivo principal (main.py)..."
cat << 'PYEOF' > main.py
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import urllib.parse

class Equipamento:
    def __init__(self, id, nome, quantidade):
        self.id = id
        self.nome = nome
        self.quantidade = quantidade

    def to_dict(self):
        return {"id": self.id, "nome": self.nome, "quantidade": self.quantidade}

banco_dados = [
    Equipamento(1, "EPI - Luva de Raspa", 50),
    Equipamento(2, "Chave de Fenda Isolada", 15)
]

class BMVCServer(BaseHTTPRequestHandler):
    def do_GET(self):
        parsed_path = urllib.parse.urlparse(self.path)
        path = parsed_path.path

        if path == "/portal":
            self.send_response(200)
            self.send_header("Content-type", "text/html; charset=utf-8")
            self.end_headers()
            html = """
            <html>
                <head><title>Portal - BMVC</title></head>
                <body style='font-family: sans-serif; padding: 20px;'>
                    <h2>Portal de Acesso - User_account: Abner da Silva Gomes (251018184)</h2>
                    <p>Autenticado com sucesso. Acesse o <a href='/banco'>Recurso Banco</a>.</p>
                </body>
            </html>
            """
            self.wfile.write(html.encode("utf-8"))

        elif path == "/banco":
            self.send_response(200)
            self.send_header("Content-type", "application/json; charset=utf-8")
            self.end_headers()
            dados = [eq.to_dict() for eq in banco_dados]
            self.wfile.write(json.dumps(dados, ensure_ascii=False).encode("utf-8"))
        else:
            self.send_response(404)
            self.end_headers()
            self.wfile.write(b"404 Not Found")

    def do_POST(self):
        if self.path == "/banco":
            content_length = int(self.headers['Content-Length'])
            post_data = self.rfile.read(content_length)
            params = json.loads(post_data.decode('utf-8'))

            novo_id = len(banco_dados) + 1
            novo_eq = Equipamento(novo_id, params.get('nome'), params.get('quantidade'))
            banco_dados.append(novo_eq)

            self.send_response(201)
            self.send_header("Content-type", "application/json; charset=utf-8")
            self.end_headers()
            self.wfile.write(json.dumps({"status": "Criado com sucesso", "item": novo_eq.to_dict()}, ensure_ascii=False).encode("utf-8"))
        else:
            self.send_response(404)
            self.
