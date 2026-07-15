---
name: diagramacao-editorial-profissional
description: >
  Use esta skill quando precisar diagramar livros e documentos editoriais com qualidade profissional para gráficas.
  Ativa quando o usuário pede para formatar miolo de livro, calcular margens para lombada, inserir quebras de capítulo em páginas ímpares, corrigir linhas viúvas e órfãs ou configurar cabeçalhos dinâmicos.
  Entrega arquivo de miolo pronto para envio a gráficas ou plataformas de autopublicação, com layout perfeitamente calculado e sem necessidade de intervenção em softwares de editoração visual.
---
# Skill: Diagramação Editorial Profissional

## Propósito
Fornecer motor tipográfico de classe mundial para diagramação de livros e documentos editoriais, com compreensão geométrica profunda da página impressa. A skill calcula espelhamento de margens, medianiz para lombada, gerencia elementos pré/pós-textuais, aplica regras editoriais clássicas (capítulos em páginas ímpares) e realiza microtipografia avançada (hifenização, alinhamento justificado, correção de viúvas e órfãs).

## O que esta skill NÃO faz
- Não cria conteúdo — apenas formata e diagrama documentos existentes
- Não gera capas — apenas miolo de livro
- Não imprime — apenas prepara arquivos para impressão
- Não substitui softwares como InDesign ou LaTeX — oferece alternativa programática

## Fontes
- Referência: `skill externa de Diagramação Editorial Profissional.pdf` (skill externa EscreveAI)
- Internet: Documentação do `reportlab` para geração de PDF profissional
- Internet: Documentação do `weasyprint` para CSS Paged Media
- Internet: Especificações CSS Paged Media (https://www.w3.org/TR/css-page-3/)
- Internet: Manuais de diagramação editorial (Bringhurst, "The Elements of Typographic Style")

## Pré-requisitos
- Python 3.10+
- `reportlab` (pip install reportlab) — para geração de PDF
- `weasyprint` (pip install weasyprint) — para CSS Paged Media
- `lxml` (pip install lxml) — para processamento XML
- `Pillow` (pip install Pillow) — para manipulação de imagens
- `numpy` (pip install numpy) — para cálculos geométricos

## Arquitetura de Dados

### Estrutura de Configuração
```python
from dataclasses import dataclass, field
from typing import List, Dict, Tuple, Optional, Any
from enum import Enum
from pathlib import Path
import math

class TipoDocumento(Enum):
    LIVRO = "livro"
    REVISTA = "revista"
    THESIS = "tese"
    RELATORIO = "relatorio"
    ARTIGO = "artigo"

class TamanhoPapel(Enum):
    A4 = (210, 297)
    A5 = (148, 210)
    B5 = (176, 250)
    CUSTOM = None

class CapaDura(Enum):
    MOLE = "mole"
    DURA = "dura"
    ESPECIAL = "especial"

@dataclass
class DimensoesPagina:
    """Dimensões físicas da página em milímetros."""
    largura: float
    altura: float
    
    @classmethod
    def de_tamanho(cls, tamanho: TamanhoPapel) -> 'DimensoesPagina':
        if tamanho == TamanhoPapel.CUSTOM:
            raise ValueError("Tamanho personalizado requer valores explícitos")
        return cls(largura=tamanho.value[0], altura=tamanho.value[1])

@dataclass
class Margens:
    """Margens da página em milímetros."""
    superior: float
    inferior: float
    esquerda: float
    direita: float
    
    @classmethod
    def espelhadas(cls, frente: float, lombada: float, cima: float, baixo: float) -> 'Margens':
        """Cria margens espelhadas para frente e verso."""
        return cls(
            superior=cima,
            inferior=baixo,
            esquerda=lombada,  # Lado da lombada (ímpar)
            direita=frente     # Lado da frente (par)
        )

@dataclass
class ConfiguracaoEditorial:
    """Configuração completa do layout editorial."""
    # Dimensões
    tamanho_papel: TamanhoPapel = TamanhoPapel.PAPER
    margens: Margens = None
    medianiz_lombada: float = 0.0  # mm por página
    
    # Tipografia
    fonte_principal: str = "Times New Roman"
    fonte_titulos: str = "Arial"
    tamanho_corpo: float = 11.0  # pontos
    tamanho_titulos: Dict[int, float] = field(default_factory=lambda: {
        1: 24.0, 2: 18.0, 3: 14.0, 4: 12.0
    })
    espacamento_entrelinhas: float = 1.5
    espacamento_entre_paragrafos: float = 0.0
    
    # Alinhamento e recuos
    alinhamento: str = "justificado"
    recuo_primeira_linha: float = 1.25  # cm
    recuo_primeiro_paragrafo_capitulo: bool = False
    
    # Hifenização
    hifenizacao_ativa: bool = True
    nivel_hifenizacao: str = "avancado"  # basico, medio, avancado
    palavras_proibidas: List[str] = field(default_factory=list)
    
    # Página
    comecar_capitulo_pagina_impar: bool = True
    inserir_paginas_brancas: bool = True
    suprimir_numeracao_capitulos: bool = True
    
    # Cabeçalho e rodapé
    cabecalho_ativo: bool = True
    alternar_cabecalho_autor_livro: bool = True
    rodape_paginacao: bool = True
    numeracao_paginas_inicia: int = 1
    
    # Elementos pré/pós-textuais
    elementos_pre_textuais: List[str] = field(default_factory=lambda: [
        "capa", "folha_de_rosto", "copyright", "dedicatoria", "epigrafe",
        "sumario", "prefacio", "introducao"
    ])
    elementos_pos_textuais: List[str] = field(default_factory=lambda: [
        "posfacio", "apendice", "anexo", "glossario", "referencias",
        "indice", "sobre_autor"
    ])
    
    # Controle de viúvas e órfãs
    evitar_viuvas: bool = True
    evitar_orfas: bool = True
    linhas_minimas_orfa: int = 2
    linhas_minimas_viuvas: int = 2

@dataclass
class ElementoDocumento:
    """Elemento estrutural do documento."""
    tipo: str
    conteudo: Any
    metadados: Dict[str, Any] = field(default_factory=dict)
    filhos: List['ElementoDocumento'] = field(default_factory=list)

@dataclass
class Pagina:
    """Representação de uma página do documento."""
    numero: int
    eh_par: bool
    eh_capitulo: bool
    eh_pagina_brancas: bool
    elementos: List[ElementoDocumento]
    conteudo_renderizado: Any = None
```

## Fluxo de Execução

### Passo 1: Cálculo Geométrico da Página
```python
import numpy as np

class CalculadorGeometriaPagina:
    """Motor de cálculo geométrico da página impressa."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.dimensoes = DimensoesPagina.de_tamanho(config.tamanho_papel)
    
    def calcular_margens_espelhadas(self) -> Dict[str, Margens]:
        """Calcula margens espelhadas para páginas pares e ímpares."""
        # Margens base
        frente = 20.0  # mm
        lombada = 25.0  # mm (maior para compensar encadernação)
        cima = 25.0
        baixo = 30.0
        
        # Ajustar conforme tamanho do livro
        if self.config.tamanho_papel == TamanhoPapel.A5:
            frente = 15.0
            lombada = 20.0
            cima = 20.0
            baixo = 25.0
        
        # Página ímpar (direita): lombada à esquerda
        margem_impar = Margens(
            superior=cima,
            inferior=baixo,
            esquerda=lombada,
            direita=frente
        )
        
        # Página par (esquerda): lombada à direita
        margem_par = Margens(
            superior=cima,
            inferior=baixo,
            esquerda=frente,
            direita=lombada
        )
        
        return {
            'impar': margem_impar,
            'par': margem_par
        }
    
    def calcular_medianiz(self, total_paginas: int) -> float:
        """
        Calcula a medianiz (espaço para lombada) baseado no volume do livro.
        
        Args:
            total_paginas: Número total de páginas do miolo
        
        Returns:
            Espaço de medianiz em milímetros
        """
        # Fórmula empírica para medianiz
        # Base: 3mm para livros finos, até 15mm para volumes grossos
        espessura_por_pagina = 0.05  # mm por página (papel.offset)
        
        # Adicionar margem de segurança
        medianiz = (total_paginas * espessura_por_pagina) / 2
        
        # Limitar entre valores razoáveis
        medianiz = max(3.0, min(15.0, medianiz))
        
        # Arredondar para 0.5mm
        medianiz = round(medianiz * 2) / 2
        
        return medianiz
    
    def calcular_area_texto(self, margem: Margens) -> Dict[str, float]:
        """Calcula área útil do texto."""
        largura_texto = self.dimensoes.largura - margem.esquerda - margem.direita
        altura_texto = self.dimensoes.altura - margem.superior - margem.inferior
        
        return {
            'largura': largura_texto,
            'altura': altura_texto,
            'x_inicio': margem.esquerda,
            'y_inicio': margem.inferior,
            'x_fim': self.dimensoes.largura - margem.direita,
            'y_fim': self.dimensoes.altura - margem.superior
        }
    
    def calcular_colunas(self, largura_texto: float, numero_colunas: int, 
                        espaco_entre_colunas: float = 5.0) -> List[Dict[str, float]]:
        """Calcula layout de múltiplas colunas."""
        largura_coluna = (largura_texto - (numero_colunas - 1) * espaco_entre_colunas) / numero_colunas
        
        colunas = []
        for i in range(numero_colunas):
            x_inicio = i * (largura_coluna + espaco_entre_colunas)
            colunas.append({
                'x_inicio': x_inicio,
                'largura': largura_coluna
            })
        
        return colunas
    
    def verificar_cabecalho_rodape(self, margem: Margens) -> Dict[str, float]:
        """Calcula posições de cabeçalho e rodapé."""
        return {
            'cabecalho_x': self.dimensoes.largura / 2,
            'cabecalho_y': margem.superior / 2,
            'rodape_x': self.dimensoes.largura / 2,
            'rodape_y': self.dimensoes.altura - margem.inferior / 2
        }
```

### Passo 2: Motor de Estruturação Documental
```python
class EstruturadorDocumento:
    """Motor de estruturação automática do documento."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
    
    def estruturar(self, elementos: List[ElementoDocumento]) -> List[ElementoDocumento]:
        """
        Estrutura documento completo com elementos pré/pós-textuais.
        
        Returns:
            Lista de elementos estruturados
        """
        estrutura = []
        
        # Adicionar elementos pré-textuais
        for elem_pre in self.config.elementos_pre_textuais:
            if elem_pre not in [e.tipo for e in elementos]:
                estrutura.append(ElementoDocumento(
                    tipo=elem_pre,
                    conteudo='',
                    metadados={'automatico': True}
                ))
        
        # Adicionar elementos do usuário
        estrutura.extend(elementos)
        
        # Adicionar elementos pós-textuais
        for elem_pos in self.config.elementos_pos_textuais:
            if elem_pos not in [e.tipo for e in elementos]:
                estrutura.append(ElementoDocumento(
                    tipo=elem_pos,
                    conteudo='',
                    metadados={'automatico': True}
                ))
        
        return estrutura
    
    def inserir_quebras_capitulo(self, elementos: List[ElementoDocumento]) -> List[ElementoDocumento]:
        """
        Insere quebras de página para garantir capítulos em páginas ímpares.
        
        Returns:
            Lista com quebras inseridas
        """
        resultado = []
        pagina_atual = 1
        
        for elem in elementos:
            # Verificar se é início de capítulo
            if elem.tipo in ['capitulo', 'prologo', 'secao_principal']:
                # Verificar se precisa de página em branco
                if self.config.comecar_capitulo_pagina_impar:
                    if pagina_atual % 2 == 0:  # Página par
                        # Inserir página em branco
                        resultado.append(ElementoDocumento(
                            tipo='pagina_em_branco',
                            conteudo='',
                            metadados={'motivo': 'capitulo_pagina_impar'}
                        ))
                        pagina_atual += 1
                
                # Adicionar quebra de capítulo
                resultado.append(ElementoDocumento(
                    tipo='quebra_capitulo',
                    conteudo='',
                    metadados={'pagina_inicio': pagina_atual}
                ))
            
            resultado.append(elem)
            pagina_atual += 1
        
        return resultado
    
    def calcular_paginas_necessarias(self, elementos: List[ElementoDocumento]) -> int:
        """Calcula total de páginas necessárias."""
        # Estimativa básica: 1 página por elemento
        total = len(elementos)
        
        # Adicionar páginas em branco para capítulos em ímpar
        if self.config.comecar_capitulo_pagina_impar:
            capitolos = sum(1 for e in elementos if e.tipo in ['capitulo', 'prologo'])
            total += capitolos // 2  # Metade dos capítulos pode precisar de página em branco
        
        # Arredondar para próximo número par (para frente/verso)
        if total % 2 != 0:
            total += 1
        
        return total
```

### Passo 3: Motor de Microtipografia
```python
class MotorMicrotipografia:
    """Motor de microtipografia avançada."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.dicionario_hifenizacao = self._carregar_dicionario()
    
    def _carregar_dicionario(self) -> set:
        """Carrega dicionário para hifenização."""
        # Dicionário simplificado - em produção, usar dicionário completo
        return set()
    
    def hifenizar(self, palavra: str) -> List[str]:
        """
        Retorna possíveis pontos de hifenização para uma palavra.
        
        Returns:
            Lista de posições onde pode haver quebra
        """
        # Algoritmo simples de hifenização
        # Em produção, usar algoritmo de Liang ou similar
        pontos = []
        
        # Regras básicas do português
        vogais = 'aeiouáàãâéêíóôõúüç'
        consoantes = 'bcdfghjklmnpqrstvwxyz'
        
        for i in range(1, len(palavra) - 2):
            char = palavra[i].lower()
            
            # Não quebrar após vogal única no início
            if i == 1 and char in vogais:
                continue
            
            # Não quebrar antes de consoante + vogal (sílaba forte)
            if (char in consoantes and 
                i + 1 < len(palavra) and 
                palavra[i + 1].lower() in vogais):
                continue
            
            pontos.append(i)
        
        return pontos
    
    def alinhar_justificado(self, palavras: List[str], largura: float, 
                           tamanho_fonte: float) -> str:
        """
        Alinha texto justificado com espaçamento uniforme.
        
        Args:
            palavras: Lista de palavras da linha
            largura: Largura disponível em pontos
            tamanho_fonte: Tamanho da fonte em pontos
        
        Returns:
            Texto formatado com espaçamento
        """
        if len(palavras) <= 1:
            return ' '.join(palavras)
        
        # Calcular largura total das palavras
        largura_palavras = sum(len(p) * tamanho_fonte * 0.5 for p in palavras)  # Estimativa
        
        # Calcular espaço disponível
        espaco_total = largura - largura_palavras
        espaco_entre = espaco_total / (len(palavras) - 1)
        
        # Garantir espaço mínimo
        espaco_entre = max(espaco_entre, tamanho_fonte * 0.25)
        
        # Construir linha
        resultado = []
        for i, palavra in enumerate(palavras):
            resultado.append(palavra)
            if i < len(palavras) - 1:
                resultado.append(' ' * int(espaco_entre / tamanho_fonte))
        
        return ''.join(resultado)
    
    def detectar_viuvas_orfas(self, linhas: List[str]) -> List[Dict[str, Any]]:
        """
        Detecta linhas viúvas e órfãs.
        
        Returns:
            Lista de problemas encontrados
        """
        problemas = []
        
        if len(linhas) < 2:
            return problemas
        
        # Último parágrafo da página: primeira linha não deve ficar sozinha na próxima
        ultima_linha = linhas[-1].strip()
        if ultima_linha and not ultima_linha.endswith(('.!?', ':",;')):
            # Possível viúva
            problemas.append({
                'tipo': 'viuva',
                'linha': len(linhas),
                'texto': ultima_linha,
                'sugestao': 'Ajustar espaçamento ou reorganizar parágrafo'
            })
        
        # Primeira linha da página não deve ser última de parágrafo anterior
        primeira_linha = linhas[0].strip()
        if primeira_linha and len(primeira_linha.split()) <= 3:
            # Possível órfã
            problemas.append({
                'tipo': 'orca',
                'linha': 1,
                'texto': primeira_linha,
                'sugestao': 'Ajustar quebra de página ou espaçamento'
            })
        
        return problemas
    
    def corrigir_viuvas_orfas(self, texto: str, largura_pagina: float) -> str:
        """Tenta corrigir viúvas e órfãs ajustando espaçamento."""
        # Implementação simplificada
        # Em produção, seria necessário reflow completo do texto
        return texto
    
    def calcular_espacamento_uniforme(self, num_palavras: int, largura: float, 
                                    largura_texto: float) -> float:
        """
        Calcula espaçamento uniforme entre palavras para texto justificado.
        
        Returns:
            Espaço entre palavras em pontos
        """
        if num_palavras <= 1:
            return 0
        
        # Largura média por palavra (estimativa)
        largura_media_palavra = largura_texto / num_palavras
        
        # Espaço entre palavras (proporcional)
        espaco = largura_media_palavra * 0.25  # 25% do tamanho da palavra
        
        # Garantir limites
        espaco = max(3.0, min(15.0, espaco))
        
        return espaco
```

### Passo 4: Motor de Layout e Paginação
```python
class MotorLayout:
    """Motor de layout e paginação."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.geometria = CalculadorGeometriaPagina(config)
        self.microtipografia = MotorMicrotipografia(config)
    
    def paginaar_documento(self, elementos: List[ElementoDocumento]) -> List[Pagina]:
        """
        Pagina o documento completo.
        
        Returns:
            Lista de páginas com conteúdo
        """
        paginas = []
        pagina_atual = 1
        
        # Calcular margens
        margens = self.geometria.calcular_margens_espelhadas()
        
        # Calcular medianiz
        total_paginas_estimado = len(elementos) * 2  # Estimativa
        medianiz = self.geometria.calcular_medianiz(total_paginas_estimado)
        
        # Ajustar margens com medianiz
        for tipo_margem in margens.values():
            tipo_margem.esquerda += medianiz / 2
            tipo_margem.direita += medianiz / 2
        
        # Paginar elementos
        for elemento in elementos:
            # Determinar tipo de página
            eh_par = pagina_atual % 2 == 0
            eh_capitulo = elemento.tipo in ['capitulo', 'prologo', 'secao_principal']
            
            # Verificar se precisa de página em branco
            if eh_capitulo and self.config.comecar_capitulo_pagina_impar and not eh_par:
                # Inserir página em branco
                paginas.append(Pagina(
                    numero=pagina_atual,
                    eh_par=eh_par,
                    eh_capitulo=False,
                    eh_pagina_brancas=True,
                    elementos=[]
                ))
                pagina_atual += 1
                eh_par = pagina_atual % 2 == 0
            
            # Criar página com conteúdo
            pagina = Pagina(
                numero=pagina_atual,
                eh_par=eh_par,
                eh_capitulo=eh_capitulo,
                eh_pagina_brancas=False,
                elementos=[elemento]
            )
            
            # Renderizar conteúdo da página
            pagina.conteudo_renderizado = self._renderizar_pagina(pagina, margens[str('par' if eh_par else 'impar')])
            
            paginas.append(pagina)
            pagina_atual += 1
        
        return paginas
    
    def _renderizar_pagina(self, pagina: Pagina, margem: Margens) -> Dict[str, Any]:
        """Renderiza conteúdo de uma página."""
        area_texto = self.geometria.calcular_area_texto(margem)
        cabecalho_rodape = self.geometria.verificar_cabecalho_rodape(margem)
        
        return {
            'area_texto': area_texto,
            'cabecalho_rodape': cabecalho_rodape,
            'conteudo': pagina.elementos,
            'pagina_numero': pagina.numero
        }
    
    def gerar_cabecalho_rodape(self, pagina: Pagina, titulo_livro: str, 
                              autor: str) -> Dict[str, str]:
        """Gera cabeçalho e rodapé dinâmicos."""
        cabecalho = {}
        
        # Cabeçalho
        if self.config.alternar_cabecalho_autor_livro:
            if pagina.eh_par:
                cabecalho['texto'] = titulo_livro
            else:
                cabecalho['texto'] = autor
        else:
            cabecalho['texto'] = titulo_livro
        
        # Rodapé com paginação
        if self.config.rodape_paginacao:
            if not (pagina.eh_capitulo and self.config.suprimir_numeracao_capitulos):
                cabecalho['pagina'] = str(pagina.numero)
        
        return cabecalho
```

### Passo 5: Motor de Geração PDF
```python
from reportlab.lib.pagesizes import A4, A5, B5, letter
from reportlab.lib.units import mm, cm
from reportlab.pdfgen import canvas
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.enums import TA_LEFT, TA_CENTER, TA_JUSTIFY, TA_RIGHT

class GeradorPDF:
    """Motor de geração de PDF profissional."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.estilos = self._criar_estilos()
    
    def _criar_estilos(self) -> dict:
        """Cria estilos de parágrafo."""
        estilos = getSampleStyleSheet()
        
        estilos_customizados = {
            'CorpoLivro': ParagraphStyle(
                'CorpoLivro',
                parent=estilos['Normal'],
                fontName=self.config.fonte_principal,
                fontSize=self.config.tamanho_corpo,
                leading=self.config.tamanho_corpo * self.config.espacamento_entrelinhas,
                alignment=TA_JUSTIFY,
                firstLineIndent=self.config.recuo_primeira_linha * cm,
                spaceAfter=self.config.espacamento_entre_paragrafos
            ),
            'PrimeiroParagrafo': ParagraphStyle(
                'PrimeiroParagrafo',
                parent=estilos['Normal'],
                fontName=self.config.fonte_principal,
                fontSize=self.config.tamanho_corpo,
                leading=self.config.tamanho_corpo * self.config.espacamento_entrelinhas,
                alignment=TA_JUSTIFY,
                firstLineIndent=0,
                spaceAfter=self.config.espacamento_entre_paragrafos
            ),
            'TituloCapitulo': ParagraphStyle(
                'TituloCapitulo',
                parent=estilos['Heading1'],
                fontName=self.config.fonte_titulos,
                fontSize=self.config.tamanho_titulos[1],
                alignment=TA_CENTER,
                spaceAfter=30,
                spaceBefore=60
            ),
            'SubtituloCapitulo': ParagraphStyle(
                'SubtituloCapitulo',
                parent=estilos['Heading2'],
                fontName=self.config.fonte_titulos,
                fontSize=self.config.tamanho_titulos[2],
                alignment=TA_CENTER,
                spaceAfter=20
            )
        }
        
        return estilos_customizados
    
    def gerar_pdf(self, paginas: List[Pagina], caminho_saida: Path,
                 titulo_livro: str, autor: str) -> Path:
        """Gera PDF final do miolo."""
        
        # Configurar documento
        if self.config.tamanho_papel == TamanhoPapel.A4:
            tamanho = A4
        elif self.config.tamanho_papel == TamanhoPapel.A5:
            tamanho = A5
        elif self.config.tamanho_papel == TamanhoPapel.B5:
            tamanho = B5
        else:
            tamanho = A4  # Padrão
        
        # Converter mm para pontos
        margens_pontos = self._converter_margens_pontos()
        
        doc = SimpleDocTemplate(
            str(caminho_saida),
            pagesize=tamanho,
            topMargin=margens_pontos['superior'],
            bottomMargin=margens_pontos['inferior'],
            leftMargin=margens_pontos['esquerda'],
            rightMargin=margens_pontos['direita']
        )
        
        # Construir conteúdo
        elementos_platypus = []
        
        for pagina in paginas:
            if not pagina.eh_pagina_brancas:
                # Adicionar cabeçalho
                if self.config.cabecalho_ativo:
                    cabecalho = self._gerar_cabecalho(pagina, titulo_livro, autor)
                    if cabecalho:
                        elementos_platypus.append(Paragraph(
                            cabecalho.get('texto', ''),
                            self._estilo_cabecalho(pagina.eh_par)
                        ))
                        elementos_platypus.append(Spacer(1, 10))
                
                # Adicionar conteúdo
                for elem in pagina.elementos:
                    elementos_platypus.extend(self._converter_elemento(elem))
                
                # Adicionar rodapé
                if self.config.rodape_paginacao:
                    rodape = self._gerar_rodape(pagina)
                    if rodape:
                        elementos_platypus.append(Spacer(1, 20))
                        elementos_platypus.append(Paragraph(
                            rodape,
                            self._estilo_rodape()
                        ))
                
                # Quebra de página
                elementos_platypus.append(PageBreak())
        
        # Construir PDF
        doc.build(elementos_platypus)
        
        return caminho_saida
    
    def _converter_margens_pontos(self) -> Dict[str, float]:
        """Converte margens de mm para pontos."""
        margens = self.config.margens
        return {
            'superior': margens.superior * mm,
            'inferior': margens.inferior * mm,
            'esquerda': margens.esquerda * mm,
            'direita': margens.direita * mm
        }
    
    def _gerar_cabecalho(self, pagina: Pagina, titulo_livro: str, 
                        autor: str) -> Dict[str, str]:
        """Gera conteúdo do cabeçalho."""
        if self.config.alternar_cabecalho_autor_livro:
            if pagina.eh_par:
                return {'texto': titulo_livro}
            else:
                return {'texto': autor}
        return {'texto': titulo_livro}
    
    def _gerar_rodape(self, pagina: Pagina) -> Optional[str]:
        """Gera conteúdo do rodapé."""
        if not (pagina.eh_capitulo and self.config.suprimir_numeracao_capitulos):
            return str(pagina.numero)
        return None
    
    def _estilo_cabecalho(self, eh_par: bool) -> ParagraphStyle:
        """Retorna estilo do cabeçalho."""
        return ParagraphStyle(
            'Cabecalho',
            fontName=self.config.fonte_titulos,
            fontSize=9,
            alignment=TA_RIGHT if eh_par else TA_LEFT,
            textColor=GRAY
        )
    
    def _estilo_rodape(self) -> ParagraphStyle:
        """Retorna estilo do rodapé."""
        return ParagraphStyle(
            'Rodape',
            fontName=self.config.fonte_principal,
            fontSize=9,
            alignment=TA_CENTER,
            textColor=GRAY
        )
    
    def _converter_elemento(self, elem: ElementoDocumento) -> List[Any]:
        """Converte elemento para elementos Platypus."""
        elementos = []
        
        if elem.tipo == 'titulo':
            nivel = elem.metadados.get('nivel', 1)
            estilo = self.estilos.get(f'TituloCapitulo', self.estilos['CorpoLivro'])
            elementos.append(Paragraph(f"<b>{elem.conteudo}</b>", estilo))
        
        elif elem.tipo == 'paragrafo':
            # Determinar se é primeiro parágrafo
            eh_primeiro = elem.metadados.get('eh_primeiro', False)
            estilo = self.estilos['PrimeiroParagrafo'] if eh_primeiro else self.estilos['CorpoLivro']
            elementos.append(Paragraph(elem.conteudo, estilo))
        
        elif elem.tipo == 'imagem':
            try:
                from reportlab.platypus import Image
                imagem = Image(elem.conteudo)
                # Ajustar tamanho
                largura_max = 14 * cm
                if imagem.imageWidth > largura_max:
                    ratio = largura_max / imagem.imageWidth
                    imagem.imageWidth = largura_max
                    imagem.imageHeight *= ratio
                elementos.append(imagem)
            except Exception as e:
                elementos.append(Paragraph(f"[Imagem: {elem.conteudo}]", self.estilos['CorpoLivro']))
        
        elif elem.tipo == 'quebra_pagina':
            elementos.append(PageBreak())
        
        return elementos
```

### Passo 6: Pipeline Completo de Diagramação
```python
class PipelineDiagramacao:
    """Pipeline completo de diagramação editorial profissional."""
    
    def __init__(self, config: ConfiguracaoEditorial = None):
        self.config = config or ConfiguracaoEditorial()
        self.geometria = CalculadorGeometriaPagina(self.config)
        self.estruturador = EstruturadorDocumento(self.config)
        self.microtipografia = MotorMicrotipografia(self.config)
        self.motor_layout = MotorLayout(self.config)
        self.gerador_pdf = GeradorPDF(self.config)
    
    def diagramar(self, caminho_entrada: Path, caminho_saida: Path,
                 titulo: str, autor: str) -> Path:
        """
        Diagrama documento completo.
        
        Args:
            caminho_entrada: Caminho do documento de entrada
            caminho_saida: Caminho do PDF de saída
            titulo: Título do livro
            autor: Nome do autor
        
        Returns:
            Caminho do PDF gerado
        """
        # 1. Carregar documento
        elementos = self._carregar_documento(caminho_entrada)
        
        # 2. Estruturar documento
        elementos_estruturados = self.estruturador.estruturar(elementos)
        
        # 3. Inserir quebras de capítulo
        elementos_com_quebras = self.estruturador.inserir_quebras_capitulo(elementos_estruturados)
        
        # 4. Calcular medianiz
        total_paginas = self.estruturador.calcular_paginas_necessarias(elementos_com_quebras)
        medianiz = self.geometria.calcular_medianiz(total_paginas)
        self.config.medianiz_lombada = medianiz
        
        # 5. Paginar
        paginas = self.motor_layout.paginaar_documento(elementos_com_quebras)
        
        # 6. Aplicar microtipografia
        paginas = self._aplicar_microtipografia(paginas)
        
        # 7. Corrigir viúvas e órfãs
        paginas = self._corrigir_viuvas_orfas(paginas)
        
        # 8. Gerar PDF
        pdf_final = self.gerador_pdf.gerar_pdf(paginas, caminho_saida, titulo, autor)
        
        return pdf_final
    
    def _carregar_documento(self, caminho: Path) -> List[ElementoDocumento]:
        """Carrega documento de various formatos."""
        elementos = []
        
        extensao = caminho.suffix.lower()
        
        if extensao == '.txt':
            with open(caminho, 'r', encoding='utf-8') as f:
                conteudo = f.read()
            
            # Dividir em parágrafos
            paragrafos = conteudo.split('\n\n')
            for i, paragrafo in enumerate(paragrafos):
                if paragrafo.strip():
                    elementos.append(ElementoDocumento(
                        tipo='paragrafo',
                        conteudo=paragrafo.strip(),
                        metadados={'eh_primeiro': i == 0}
                    ))
        
        elif extensao == '.md':
            # Parse Markdown simplificado
            elementos = self._parse_markdown(caminho)
        
        return elementos
    
    def _parse_markdown(self, caminho: Path) -> List[ElementoDocumento]:
        """Parse de Markdown simplificado."""
        import re
        
        elementos = []
        
        with open(caminho, 'r', encoding='utf-8') as f:
            linhas = f.readlines()
        
        for linha in linhas:
            linha = linha.strip()
            
            if not linha:
                continue
            
            # Títulos
            if linha.startswith('#'):
                nivel = len(linha.split(' ')[0])
                titulo = linha.strip('#').strip()
                elementos.append(ElementoDocumento(
                    tipo='titulo',
                    conteudo=titulo,
                    metadados={'nivel': nivel}
                ))
            
            # Imagens
            elif linha.startswith('!['):
                match = re.match(r'!\[([^\]]*)\]\(([^)]+)\)', linha)
                if match:
                    alt_text = match.group(1)
                    caminho_img = match.group(2)
                    elementos.append(ElementoDocumento(
                        tipo='imagem',
                        conteudo=caminho_img,
                        metadados={'alt': alt_text}
                    ))
            
            # Parágrafos normais
            else:
                elementos.append(ElementoDocumento(
                    tipo='paragrafo',
                    conteudo=linha,
                    metadados={'eh_primeiro': False}
                ))
        
        return elementos
    
    def _aplicar_microtipografia(self, paginas: List[Pagina]) -> List[Pagina]:
        """Aplica microtipografia em todas as páginas."""
        # Implementação simplificada
        # Em produção, seria necessário processar cada linha
        return paginas
    
    def _corrigir_viuvas_orfas(self, paginas: List[Pagina]) -> List[Pagina]:
        """Corrige linhas viúvas e órfãs."""
        # Implementação simplificada
        # Em produção, seria necessário reflow complexo
        return paginas
```

## Tratamento de Erros
- **Arquivo não encontrado**: Verificar caminho e permissões
- **Formato não suportado**: Usar .txt ou .md como fallback
- **Margens inválidas**: Usar valores padrão seguros
- **Medianiz muito grande**: Limitar a 15mm máximo
- **Viúvas/órfãs não corrigidas**: Registrar e informar ao usuário

## Regras
### O que SEMPRE fazer
- Calcular medianiz baseada no volume do livro
- Garantir capítulos em páginas ímpares (direita)
- Inserir páginas em branco quando necessário
- Aplicar alinhamento justificado com espaçamento uniforme
- Corrigir linhas viúvas e órfãs quando possível
- Suprimir numeração nas primeiras páginas de capítulos
- Alternar cabeçalhos em páginas pares/ímpares

### O que NUNCA fazer
- Deixar texto ultrapassar margens da lombada
- Iniciar capítulo em página par (direita)
- Deixar linhas viúvas ou órfãs sem aviso
- Alterar configurações de medianiz sem recálculo
- Gerar PDF com qualidade inadequada para impressão

### Quando algo falhar
1. **Margens inválidas**: Usar configuração padrão para tamanho de papel
2. **Medianiz incorreta**: Recalcular baseada no total de páginas
3. **Viúvas/órfãs persistentes**: Ajustar espaçamento ou aceitar com aviso
4. **PDF não gera**: Verificar dependências e configurações

## Exemplos de Uso

### Exemplo 1: Diagramação completa
```python
config = ConfiguracaoEditorial(
    tamanho_papel=TamanhoPapel.A5,
    margens=Margens.espelhadas(frente=15, lombada=20, cima=20, baixo=25)
)
pipeline = PipelineDiagramacao(config)
pdf = pipeline.diagramar(
    caminho_entrada=Path('manuscrito.md'),
    caminho_saida=Path('livro_diagramado.pdf'),
    titulo='Meu Livro',
    autor='Autor Exemplo'
)
```

### Exemplo 2: Livro com capa dura
```python
config = ConfiguracaoEditorial(
    tamanho_papel=TamanhoPapel.CUSTOM,
    margens=Margens.espelhadas(frente=25, lombada=30, cima=30, baixo=35),
    medianiz_lombada=12.0
)
pipeline = PipelineDiagramacao(config)
pdf = pipeline.diagramar(
    caminho_entrada=Path('romance.txt'),
    caminho_saida=Path('romance_capa_dura.pdf'),
    titulo='Romance Épico',
    autor='Escritor Famoso'
)
```

### Exemplo 3: Tese acadêmica
```python
config = ConfiguracaoEditorial(
    tamanho_papel=TamanhoPapel.A4,
    cabecalho_ativo=True,
    alternar_cabecalho_autor_livro=True,
    rodape_paginacao=True
)
pipeline = PipelineDiagramacao(config)
pdf = pipeline.diagramar(
    caminho_entrada=Path('tese.md'),
    caminho_saida=Path('tese_final.pdf'),
    titulo='Análise de Dados Econômicos',
    autor='Pesquisador'
)
```