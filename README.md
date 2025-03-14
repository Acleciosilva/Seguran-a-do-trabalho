import pyautogui
import pytesseract
from PIL import Image
import time

# Capturar a tela do celular
def capturar_tela():
    screenshot = pyautogui.screenshot()
    screenshot.save("/storage/emulated/0/Download/tela.png")  # Salva na pasta Downloads
    return screenshot

# Extrair texto da imagem
def extrair_texto():
    imagem = Image.open("/storage/emulated/0/Download/tela.png")
    texto = pytesseract.image_to_string(imagem, lang="eng")  # Reconhece números
    return texto

# Loop para capturar a tela e analisar o multiplicador
while True:
    capturar_tela()
    texto_extraido = extrair_texto()
    print("Texto encontrado:", texto_extraido)

    # Se encontrar um multiplicador acima de 2.0, simula um clique na aposta
    if "2.00" in texto_extraido or "3.00" in texto_extraido:
        pyautogui.click(x=500, y=600)  # Coordenadas do botão de aposta (ajustar)
        print("Aposta realizada automaticamente!")

    time.sleep(3)  # Aguarda 3 segundos antes de capturar novamente
