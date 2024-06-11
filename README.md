import tkinter as tk
from tkinter import messagebox, filedialog
import sqlite3
from pdf2docx import Converter
from docx2pdf import convert as convert_to_pdf
from PIL import Image, ImageTk
import os

conn = sqlite3.connect('usuarios.db')
cursor = conn.cursor()

cursor.execute('''
    CREATE TABLE IF NOT EXISTS usuarios (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        usuario TEXT NOT NULL,
        senha TEXT NOT NULL
    )
''')
conn.commit()

def verificar_login():
    usuario = entrada_usuario.get()
    senha = entrada_senha.get()

    cursor.execute("SELECT * FROM usuarios WHERE usuario=? AND senha=?", (usuario, senha))
    resultado = cursor.fetchone()

    if resultado:
        abrir_janela_principal()
    else:
        messagebox.showerror("Erro", "Usuário ou senha incorretos.")

def registrar_usuario():
    usuario = entrada_novo_usuario.get()
    senha = entrada_nova_senha.get()

    if usuario and senha:
        cursor.execute("INSERT INTO usuarios (usuario, senha) VALUES (?, ?)", (usuario, senha))
        conn.commit()
        messagebox.showinfo("Sucesso", "Usuário registrado com sucesso!")
        janela_registro.destroy()
    else:
        messagebox.showerror("Erro", "Todos os campos são obrigatórios.")

def abrir_janela_registro():
    global janela_registro
    janela_registro = tk.Toplevel(janela)
    janela_registro.title("Registro de Novo Usuário")
    janela_registro.geometry("300x200")
    janela_registro.resizable(False, False)

    frame_registro = tk.Frame(janela_registro, pady=10)
    frame_registro.pack()

    label_novo_usuario = tk.Label(frame_registro, text="Novo Usuário:", font=("Helvetica", 12))
    label_novo_usuario.grid(row=0, column=0, padx=10, pady=5, sticky=tk.W)
    global entrada_novo_usuario
    entrada_novo_usuario = tk.Entry(frame_registro, font=("Helvetica", 12))
    entrada_novo_usuario.grid(row=0, column=1, padx=10, pady=5)

    label_nova_senha = tk.Label(frame_registro, text="Nova Senha:", font=("Helvetica", 12))
    label_nova_senha.grid(row=1, column=0, padx=10, pady=5, sticky=tk.W)
    global entrada_nova_senha
    entrada_nova_senha = tk.Entry(frame_registro, font=("Helvetica", 12), show="*")
    entrada_nova_senha.grid(row=1, column=1, padx=10, pady=5)

    botao_registrar = tk.Button(frame_registro, text="Registrar", command=registrar_usuario, bg="blue", fg="white", font=("Helvetica", 12))
    botao_registrar.grid(row=2, columnspan=2, pady=10)

def abrir_janela_principal():
    janela.withdraw()
    janela_principal = tk.Toplevel()
    janela_principal.title("Conversor de Arquivos")
    janela_principal.geometry("780x520")
    janela_principal.resizable(False, False)

    bg_image_path = "C:/Users/Enzo/Pictures/Saved Pictures/images.png"
    bg_image = Image.open(bg_image_path)
    bg_image = bg_image.resize((780, 520), Image.LANCZOS)
    bg_photo = ImageTk.PhotoImage(bg_image)

    canvas = tk.Canvas(janela_principal, width=780, height=520)
    canvas.pack(fill="both", expand=True)
    canvas.create_image(0, 0, image=bg_photo, anchor="nw")
    canvas.image = bg_photo  # Manter referência

    def create_transparent_button(x, y, text, command, bg_color, fg_color):
        button = tk.Button(canvas, text=text, command=command, bg=bg_color, fg=fg_color, font=("Helvetica", 12))
        canvas.create_window(x, y, anchor="nw", window=button)

    def converter_para_pdf_e_baixar():
        caminho_word = filedialog.askopenfilename(filetypes=[("Arquivos Word", "*.docx")])
        if caminho_word:
            try:
                caminho_pdf = caminho_word.replace('.docx', '.pdf')
                convert_to_pdf(caminho_word, caminho_pdf)
                messagebox.showinfo("Sucesso", f"Arquivo convertido para {caminho_pdf}")
                baixar_arquivo(caminho_pdf)
            except Exception as e:
                messagebox.showerror("Erro", f"Falha na conversão: {e}")

    def converter_para_word_e_baixar():
        caminho_pdf = filedialog.askopenfilename(filetypes=[("Arquivos PDF", "*.pdf")])
        if caminho_pdf:
            caminho_word = caminho_pdf.replace('.pdf', '.docx')
            cv = Converter(caminho_pdf)
            cv.convert(caminho_word)
            cv.close()
            messagebox.showinfo("Sucesso", f"Arquivo convertido para {caminho_word}")
            baixar_arquivo(caminho_word)

    def baixar_arquivo(caminho_arquivo):
        if caminho_arquivo:
            nome_arquivo = filedialog.asksaveasfilename(defaultextension="*.*", title="Salvar arquivo como")
            if nome_arquivo:
                try:
                    with open(caminho_arquivo, 'rb') as file:
                        conteudo = file.read()
                    with open(nome_arquivo, 'wb') as file:
                        file.write(conteudo)
                    messagebox.showinfo("Sucesso", f"Arquivo baixado como {nome_arquivo}")
                except Exception as e:
                    messagebox.showerror("Erro", f"Falha no download do arquivo: {e}")

    def converter_jpeg_para_png():
        caminho_jpeg = filedialog.askopenfilename(filetypes=[("Arquivos JPEG", "*.jpg;*.jpeg")])
        if caminho_jpeg:
            caminho_png = caminho_jpeg.rsplit('.', 1)[0] + '.png'
            imagem = Image.open(caminho_jpeg)
            imagem.save(caminho_png)
            messagebox.showinfo("Sucesso", f"Arquivo convertido para {caminho_png}")
            baixar_arquivo(caminho_png)

    def converter_png_para_jpeg():
        caminho_png = filedialog.askopenfilename(filetypes=[("Arquivos PNG", "*.png")])
        if caminho_png:
            caminho_jpeg = caminho_png.rsplit('.', 1)[0] + '.jpeg'
            imagem = Image.open(caminho_png)
            imagem.save(caminho_jpeg)
            messagebox.showinfo("Sucesso", f"Arquivo convertido para {caminho_jpeg}")
            baixar_arquivo(caminho_jpeg)

    create_transparent_button(150, 100, "Word para PDF", converter_para_pdf_e_baixar, "blue", "white")
    create_transparent_button(400, 100, "PDF para Word", converter_para_word_e_baixar, "green", "white")
    create_transparent_button(150, 200, "JPEG para PNG", converter_jpeg_para_png, "purple", "white")
    create_transparent_button(400, 200, "PNG para JPEG", converter_png_para_jpeg, "orange", "white")

    botao_sair = tk.Button(janela_principal, text="Sair", command=janela_principal.destroy, bg="red", fg="white", font=("Helvetica", 12))
    canvas.create_window(340, 300, anchor="nw", window=botao_sair)

def fechar_aplicacao():
    conn.close()
    janela.destroy()

janela = tk.Tk()
janela.title("Tela de Login")
janela.geometry("480x360")
janela.resizable(False, False)

frame_titulo = tk.Frame(janela, bg="red")
frame_titulo.pack(fill=tk.X)

titulo = tk.Label(frame_titulo, text="Login", bg="red", fg="black", font=("Helvetica", 20))
titulo.pack(pady=10)

frame_entrada = tk.Frame(janela, pady=10)
frame_entrada.pack()

label_usuario = tk.Label(frame_entrada, text="Usuário:", font=("Helvetica", 12))
label_usuario.grid(row=0, column=0, padx=10, pady=5, sticky=tk.W)
entrada_usuario = tk.Entry(frame_entrada, font=("Helvetica", 12))
entrada_usuario.grid(row=0, column=1, padx=10, pady=5)

label_senha = tk.Label(frame_entrada, text="Senha:", font=("Helvetica", 12))
label_senha.grid(row=1, column=0, padx=10, pady=5, sticky=tk.W)
entrada_senha = tk.Entry(frame_entrada, font=("Helvetica", 12), show="*")
entrada_senha.grid(row=1, column=1, padx=10, pady=5)

frame_botoes = tk.Frame(janela, pady=10)
frame_botoes.pack()

botao_login = tk.Button(frame_botoes, text="Login", command=verificar_login, bg="green", fg="white", font=("Helvetica", 12))
botao_login.grid(row=0, column=0, padx=10)

botao_registrar = tk.Button(frame_botoes, text="Registrar", command=abrir_janela_registro, bg="orange", fg="white", font=("Helvetica", 12))
botao_registrar.grid(row=0, column=1, padx=10)

botao_sair = tk.Button(frame_botoes, text="Sair", command=fechar_aplicacao, bg="red", fg="white", font=("Helvetica", 12))
botao_sair.grid(row=0, column=2, padx=10)

janela.mainloop()


Alunos: Enzo Bisco Padovani, RA- 202102074703
        João Pedro Rocha Santos, RA- 202302380344
        Mateus Cardoso Terra, RA- 202202140422


Apresentação do Trabalho
Este projeto consiste em um aplicativo de conversão de arquivos, desenvolvido com a biblioteca Tkinter para a interface gráfica e várias outras bibliotecas para manipulação de arquivos.
O objetivo principal do aplicativo é fornecer uma interface intuitiva para converter arquivos entre diferentes formatos, como Word para PDF, PDF para Word, JPEG para PNG e PNG para JPEG.
Além disso, o sistema inclui uma tela de login e registro de usuários, com armazenamento de dados em um banco de dados SQLite.

Funcionalidades
Login e Registro de Usuários: O aplicativo permite que usuários façam login ou se registrem para acessar a funcionalidade de conversão de arquivos.
Conversão de Arquivos:
Word para PDF: Utiliza a biblioteca docx2pdf para converter documentos Word (.docx) para PDF.
PDF para Word: Utiliza a biblioteca pdf2docx para converter arquivos PDF para Word (.docx).
JPEG para PNG: Utiliza a biblioteca PIL (Pillow) para converter imagens JPEG para PNG.
PNG para JPEG: Utiliza a biblioteca PIL (Pillow) para converter imagens PNG para JPEG.
Bibliotecas Utilizadas
Tkinter: Biblioteca padrão do Python para a criação de interfaces gráficas.
pip install tk (Geralmente já está incluída na instalação padrão do Python)
sqlite3: Biblioteca padrão do Python para interação com bancos de dados SQLite.
Não requer instalação adicional.
docx2pdf: Biblioteca para converter documentos Word para PDF.
pip install docx2pdf
pdf2docx: Biblioteca para converter arquivos PDF para Word.
pip install pdf2docx
Pillow (PIL): Biblioteca para manipulação de imagens.
pip install pillow
requests: Biblioteca para fazer requisições HTTP. (Nota: Embora mencionada no código original, não foi utilizada na versão final)
pip install requests
