import re
import docx
import PyPDF2

# ----------------- FUNÇÕES DE LEITURA -----------------
def ler_docx(caminho):
    doc = docx.Document(caminho)
    texto = "\n".join([p.text for p in doc.paragraphs])
    return texto

def ler_pdf(caminho):
    texto = ""
    with open(caminho, "rb") as f:
        leitor = PyPDF2.PdfReader(f)
        for pagina in leitor.pages:
            texto += pagina.extract_text() + "\n"
    return texto

# ----------------- ANÁLISE DE CURRÍCULO -----------------
def analisar_curriculo(texto, vaga):
    resultado = {}

    # Nome
    linhas = texto.split("\n")
    resultado["nome"] = linhas[0].strip() if linhas[0] else "Não identificado"

    # Formação mínima
    if "ensino médio" in texto.lower():
        resultado["formacao"] = ["Ensino Médio"]
    elif "técnico" in texto.lower():
        resultado["formacao"] = ["Curso Técnico"]
    else:
        resultado["formacao"] = ["Não identificada"]

    # Experiência
    anos_exp = re.findall(r"(\d+)\s+anos", texto.lower())
    resultado["anos_experiencia"] = int(anos_exp[0]) if anos_exp else 0

    # Cursos e habilidades extras
    cursos_extras = ["excel", "word", "powerpoint", "pacote office", "inglês", "atendimento", "vendas"]
    soft_skills = ["comunicação", "trabalho em equipe", "organização", "pontualidade", "criatividade", "liderança"]

    encontrados_cursos = [c for c in cursos_extras if c in texto.lower()]
    encontrados_soft = [s for s in soft_skills if s in texto.lower()]

    resultado["cursos_extras"] = encontrados_cursos
    resultado["soft_skills"] = encontrados_soft

    # Avaliação
    dicas = []
    aprovado = True

    if resultado["anos_experiencia"] == 0:
        dicas.append("Mesmo sem experiência formal, destaque trabalhos voluntários, escolares ou informais.")
        aprovado = True  # não reprovar só por falta de experiência

    if resultado["formacao"] == ["Não identificada"]:
        dicas.append("Adicione sua escolaridade (Ensino Médio, Técnico ou curso superior em andamento).")
        aprovado = False

    if not encontrados_cursos:
        dicas.append("Faça cursos online gratuitos (ex.: Excel, Atendimento, Inglês) para fortalecer seu currículo.")
        aprovado = False

    if not encontrados_soft:
        dicas.append("Inclua habilidades pessoais (ex.: Comunicação, Trabalho em equipe, Organização).")
        aprovado = False

    if not dicas:
        dicas.append("Parabéns! Seu currículo já está bem estruturado para uma vaga de primeiro emprego.")

    resultado["aprovado"] = aprovado
    resultado["dicas"] = dicas

    return resultado


# ----------------- LISTA DE VAGAS -----------------
vagas = [
    {
        "titulo": "Atendente de Loja",
        "requisitos": ["comunicação", "atendimento", "organização"],
        "anos_experiencia_min": 0,
        "formacao": ["ensino médio", "curso técnico"]
    },
    {
        "titulo": "Auxiliar Administrativo",
        "requisitos": ["excel", "word", "organização"],
        "anos_experiencia_min": 0,
        "formacao": ["ensino médio", "curso técnico"]
    }
]

# ----------------- ESCOLHER ARQUIVO -----------------
# Altere o caminho para o currículo que deseja analisar
arquivo = "curriculo.docx"   # pode ser "curriculo.pdf" também

if arquivo.endswith(".docx"):
    curriculo_texto = ler_docx(arquivo)
elif arquivo.endswith(".pdf"):
    curriculo_texto = ler_pdf(arquivo)
else:
    raise ValueError("Formato de arquivo não suportado. Use .docx ou .pdf")

# ----------------- ANÁLISE -----------------
print("Análise de Currículo para Vagas de Primeiro Emprego")
print("===================================================")

for vaga in vagas:
    print(f"\n--- Vaga: {vaga['titulo']} ---")
    resultado = analisar_curriculo(curriculo_texto, vaga)

    print(f"Nome: {resultado['nome']}")
    print(f"Formação: {', '.join(resultado['formacao'])}")
    print(f"Anos de Experiência: {resultado['anos_experiencia']}")
    print(f"Cursos Extras: {', '.join(resultado['cursos_extras']) if resultado['cursos_extras'] else 'Nenhum'}")
    print(f"Soft Skills: {', '.join(resultado['soft_skills']) if resultado['soft_skills'] else 'Nenhuma'}")
    print("\nDicas:")
    for dica in resultado["dicas"]:
        print(f"- {dica}")
    print("\nStatus Final:", "APROVADO ✅" if resultado["aprovado"] else "EM DESENVOLVIMENTO ⚠️")
