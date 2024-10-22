# PDFTOOLS
pdfTools é uma ferramenta de linha de comando desenvolvida em Python para dividir arquivos PDF em partes menores com base em um tamanho máximo especificado. Utilizando a biblioteca `pikepdf`, o pdfTools permite que você gerencie e organize documentos PDF de forma eficiente, facilitando o manuseio de arquivos grandes ou a necessidade de dividir documentos para compartilhamento.

## 📄 **Recursos**

- **Divisão por Tamanho:** Separe um PDF grande em múltiplos arquivos menores, cada um com um tamanho máximo definido.
- **Compatibilidade:** Suporta a maioria dos arquivos PDF, garantindo a integridade das páginas durante o processo de divisão.
- **Fácil de Usar:** Interface simples de linha de comando para operações rápidas e eficientes.

## 🚀 **Começando**

Estas instruções irão ajudá-lo a configurar e utilizar o pdfTools em seu ambiente local para fins de desenvolvimento e teste.

### 🔧 **Pré-requisitos**

Antes de começar, certifique-se de ter o seguinte instalado:

- **Python 3.6 ou superior:** [Download Python](https://www.python.org/downloads/)
- **pip:** Geralmente incluído com o Python. Caso não tenha, siga as instruções [aqui](https://pip.pypa.io/en/stable/installation/).

### 📦 **Instalação**

1. **Clone o Repositório:**

   ```bash
   git clone https://github.com/seu-usuario/pdfTools.git
   ```

2. **Navegue até o Diretório do Projeto:**

   ```bash
   cd pdfTools
   ```

3. **Crie um Ambiente Virtual (Opcional, mas recomendado):**

   ```bash
   python -m venv venv
   ```

4. **Ative o Ambiente Virtual:**

   - **No Windows:**

     ```bash
     venv\Scripts\activate
     ```

   - **No Unix ou MacOS:**

     ```bash
     source venv/bin/activate
     ```

5. **Instale as Dependências Necessárias:**

   ```bash
   pip install -r requirements.txt
   ```

   **Nota:** Se não houver um arquivo `requirements.txt`, você pode instalar diretamente o `pikepdf`:

   ```bash
   pip install pikepdf
   ```

### 📜 **Uso**

O pdfTools é executado via linha de comando. A seguir, você encontrará instruções sobre como utilizar a ferramenta para dividir seus arquivos PDF.

1. **Prepare seu PDF:**

   Certifique-se de que o arquivo PDF que você deseja dividir está acessível e anote seu caminho completo.

2. **Execute o Script:**

   ```bash
   python dividirPdfs.py
   ```

3. **Configuração de Parâmetros:**

   No script `dividirPdfs.py`, você encontrará a seção `if __name__ == "__main__":` onde você pode configurar os seguintes parâmetros:

   - **`caminho_pdf`:** Caminho completo para o arquivo PDF original que você deseja dividir.
   - **`diretorio_saida`:** Diretório onde os PDFs divididos serão salvos. Se o diretório não existir, ele será criado automaticamente.
   - **`tamanho_max_mb`:** Tamanho máximo em megabytes para cada PDF dividido.

   **Exemplo:**

   ```python
   if __name__ == "__main__":
       # Exemplo de uso
       caminho_pdf = r"C:\Documentos\Original.pdf"       # Substitua pelo caminho do seu PDF
       diretorio_saida = r"C:\Documentos\PDFs_Divididos" # Substitua pelo diretório de saída desejado
       tamanho_max_mb = 10                                # Defina o tamanho máximo em MB

       dividir_pdf_por_tamanho_pikepdf(caminho_pdf, diretorio_saida, tamanho_max_mb)
   ```

4. **Executar o Script com Parâmetros:**

   Após configurar os parâmetros no script, execute novamente:

   ```bash
   python dividirPdfs.py
   ```

   **Saída Esperada:**

   ```
   Processando página 1: Tamanho atual = 0.04 MB
   Processando página 2: Tamanho atual = 0.05 MB
   ...
   Salvo: parte_1.pdf
   Iniciando nova parte com a página 91.
   Salvo: parte_2.pdf
   Divisão concluída!
   ```

### 🎯 **Exemplo Completo do Script**

Abaixo está o script completo `dividirPdfs.py` atualizado para remover a última página corretamente usando índices positivos:

```python
import os
from pikepdf import Pdf, PdfError

def dividir_pdf_por_tamanho_pikepdf(caminho_pdf, diretorio_saida, tamanho_max_mb):
    """
    Divide um PDF em vários PDFs com tamanho máximo especificado usando pikepdf.

    :param caminho_pdf: Caminho para o PDF original.
    :param diretorio_saida: Diretório onde os PDFs divididos serão salvos.
    :param tamanho_max_mb: Tamanho máximo de cada PDF dividido em megabytes.
    """
    # Garantir que o diretório de saída exista
    os.makedirs(diretorio_saida, exist_ok=True)

    try:
        pdf = Pdf.open(caminho_pdf)
    except PdfError as e:
        print(f"Erro ao abrir o PDF: {e}")
        return

    parte = 1
    escritor = Pdf.new()
    for i, pagina in enumerate(pdf.pages, start=1):
        escritor.pages.append(pagina)
        # Salvar temporariamente e verificar o tamanho
        temp_path = os.path.join(diretorio_saida, f"parte_{parte}_temp.pdf")
        try:
            escritor.save(temp_path)
        except PdfError as e:
            print(f"Erro ao salvar temporariamente o PDF: {e}")
            return

        tamanho_atual_mb = os.path.getsize(temp_path) / (1024 * 1024)
        print(f"Processando página {i}: Tamanho atual = {tamanho_atual_mb:.2f} MB")

        if tamanho_atual_mb > tamanho_max_mb:
            if len(escritor.pages) == 1:
                print(f"A página {i} excede o tamanho máximo de {tamanho_max_mb} MB e será salva individualmente.")
                try:
                    escritor.save(os.path.join(diretorio_saida, f"parte_{parte}.pdf"))
                    print(f"Salvo: parte_{parte}.pdf")
                except PdfError as e:
                    print(f"Erro ao salvar o PDF 'parte_{parte}.pdf': {e}")
                    return
                parte += 1
                escritor = Pdf.new()
            else:
                # Remover a última página adicionada usando o índice positivo
                try:
                    indice_ultima_pagina = len(escritor.pages) - 1
                    escritor.pages.remove(p=indice_ultima_pagina)
                    print(f"Removida a página {i} do escritor para salvar a parte anterior.")
                except PdfError as e:
                    print(f"Erro ao remover a página {i}: {e}")
                    return

                # Salvar o PDF atual sem a última página
                try:
                    escritor.save(os.path.join(diretorio_saida, f"parte_{parte}.pdf"))
                    print(f"Salvo: parte_{parte}.pdf")
                except PdfError as e:
                    print(f"Erro ao salvar o PDF 'parte_{parte}.pdf': {e}")
                    return
                parte += 1

                # Reiniciar o escritor e adicionar a página que excedeu
                escritor = Pdf.new()
                escritor.pages.append(pagina)
                print(f"Iniciando nova parte com a página {i}.")

        # Remover o arquivo temporário
        if os.path.exists(temp_path):
            os.remove(temp_path)

    # Salvar qualquer conteúdo restante
    if len(escritor.pages) > 0:
        try:
            escritor.save(os.path.join(diretorio_saida, f"parte_{parte}.pdf"))
            print(f"Salvo: parte_{parte}.pdf")
        except PdfError as e:
            print(f"Erro ao salvar o PDF 'parte_{parte}.pdf': {e}")
            return

    print("Divisão concluída!")

if __name__ == "__main__":
    # Exemplo de uso
    caminho_pdf = r"F:\intp\Dropbox\INTP\JURÍDICO\01 - CONTENCIOSO\Casos\41 - MS - Esteio\processoIntegra\Fichário1.pdf"       # Substitua pelo caminho do seu PDF
    diretorio_saida = r"F:\intp\Dropbox\INTP\JURÍDICO\01 - CONTENCIOSO\Casos\41 - MS - Esteio\processoIntegra\pdfs_divididos"                # Substitua pelo diretório de saída desejado
    tamanho_max_mb = 10                                # Defina o tamanho máximo em MB

    dividir_pdf_por_tamanho_pikepdf(caminho_pdf, diretorio_saida, tamanho_max_mb)
```

## 🛠️ **Contribuindo**

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests para melhorar o pdfTools.

### 🔄 **Atualizações**

1. **Fork o Repositório:**

   Clique no botão "Fork" no canto superior direito da página do GitHub para criar uma cópia do repositório.

2. **Clone o Repositório Forked:**

   ```bash
   git clone https://github.com/hqr90/pdfTools.git
   cd pdfTools
   ```

3. **Crie uma Branch para sua Feature ou Correção:**

   ```bash
   git checkout -b minha-nova-feature
   ```

4. **Faça suas Alterações e Commit:**

   ```bash
   git add .
   git commit -m "Descrição clara da sua alteração"
   ```

5. **Envie para o Repositório Forked:**

   ```bash
   git push origin minha-nova-feature
   ```

6. **Abra um Pull Request:**

   Vá até o repositório original no GitHub e abra um Pull Request a partir da sua branch.

## 📄 **Licença**

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 📞 **Contato**

Se você tiver quaisquer dúvidas ou sugestões, sinta-se à vontade para entrar em contato:

- **Email:** rebello.hiltonqueiroz@gmail.com
- **GitHub:** [@seu-usuario](https://github.com/hqr90)

---

📝 **Nota:** Este projeto está em constante desenvolvimento. Fique atento para futuras atualizações e melhorias!
