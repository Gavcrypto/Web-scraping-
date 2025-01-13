from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# Configurar o serviço do ChromeDriver
servico = Service(ChromeDriverManager().install())
navegador = webdriver.Chrome(service=servico)

def extrair_dados_ativo(navegador, ativo, data_referencia):
    """
    Função para acessar a página do ativo, localizar a tabela e extrair o valor de fechamento.
    """
    # Construir a URL do ativo
    url = f"https://www.coingecko.com/en/coins/{ativo.lower().replace(' ', '-')}/historical_data"
    navegador.get(url)
    
    try:
        # Esperar que a tabela carregue
        WebDriverWait(navegador, 20).until(
            EC.presence_of_element_located((By.XPATH, "/html/body/div[2]/main/div[2]"))
        )

        # Localizar a tabela
        tabela = navegador.find_element(By.XPATH, "/html/body/div[2]/main/div[2]")

        # Localizar todas as linhas da tabela
        linhas = tabela.find_elements(By.TAG_NAME, "tr")

        # Iterar pelas linhas para encontrar a data e o valor de fechamento
        for linha in linhas:
            colunas = linha.find_elements(By.TAG_NAME, "td")
            if colunas and colunas[0].text.strip() == data_referencia:  # Verificar a data na primeira coluna
                valor_fechamento = colunas[4].text.strip()  # A coluna "Close" está no índice 4
                return f"{ativo.upper()} {valor_fechamento}"
        return f"{ativo.upper()} N/A"  # Caso a data não seja encontrada
    except:
        return f"{ativo.upper()} N/A"  # Retorna N/A em caso de erro


# Lista completa de ativos
ativos = [
    "Bitcoin", "Ethereum", "Solana", "Cardano"]
    
# Data de referência
data_referencia = "2025-01-12"

# Processar cada ativo
try:
    for index, ativo in enumerate(ativos):
        if index > 0:  # Para os ativos após o primeiro, abrir uma nova aba
            navegador.execute_script("window.open('');")
            navegador.switch_to.window(navegador.window_handles[-1])  # Mudar para a nova aba

        # Extrair e imprimir os dados do ativo atual
        print(extrair_dados_ativo(navegador, ativo, data_referencia))

        # Pausa entre as pesquisas (ajuste conforme necessário)
        time.sleep(5)

        # Fechar a aba anterior, se houver
        if len(navegador.window_handles) > 1:
            navegador.switch_to.window(navegador.window_handles[0])
            navegador.close()

        # Voltar para a aba atual
        navegador.switch_to.window(navegador.window_handles[0])

except Exception as e:
    print(f"Ocorreu um erro geral: {e}")

# O navegador permanecerá aberto
print("Navegador permanecerá aberto. Feche manualmente quando terminar.")
