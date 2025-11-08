# Roteiro Completo para Desenvolvimento de Software Contábil em Python

## 📋 Índice do Roteiro
1. [Fase 1: Planejamento e Análise](#fase-1-planejamento-e-análise)
2. [Fase 2: Ambiente de Desenvolvimento](#fase-2-ambiente-de-desenvolvimento)
3. [Fase 3: Desenvolvimento por Módulos](#fase-3-desenvolvimento-por-módulos)
4. [Fase 4: Módulos Principais](#fase-4-módulos-principais)
5. [Fase 5: Funcionalidades Avançadas](#fase-5-funcionalidades-avançadas)
6. [Fase 6: Testes e Validação](#fase-6-testes-e-validação)
7. [Fase 7: Implantação e Documentação](#fase-7-implantação-e-documentação)
8. [Stack Tecnológica](#stack-tecnológica)
9. [Cronograma](#cronograma)
10. [Dicas Importantes](#dicas-importantes)

---

## Fase 1: Planejamento e Análise (1-2 semanas)

### 1.1 Definição de Requisitos

**Módulos Principais Necessários:**
```python
MODULOS_PRINCIPAIS = [
    "Cadastro de Clientes/Fornecedores",
    "Plano de Contas",
    "Lançamentos Contábeis",
    "Relatórios (Balanço, DRE, Razão)",
    "Apuração de Impostos",
    "Conciliação Bancária",
    "Contas a Pagar/Receber",
    "Folha de Pagamento"
]
```

**Requisitos Funcionais:**
- Cadastro completo de empresas e usuários
- Gestão do plano de contas contábil
- Sistema de lançamentos com partidas dobradas
- Geração de relatórios fiscais e contábeis
- Cálculo automático de impostos
- Conciliação bancária integrada

**Requisitos Não-Funcionais:**
- Segurança de dados sensíveis
- Backup automático
- Interface intuitiva
- Performance em grandes volumes de dados

### 1.2 Escolha das Tecnologias

**Backend:** Python + Django
**Database:** PostgreSQL
**Frontend:** HTML5 + CSS3 + JavaScript + Bootstrap
**Relatórios:** ReportLab + Pandas
**Gráficos:** Chart.js
**Deploy:** Heroku (free tier) ou servidor próprio

---

## Fase 2: Ambiente de Desenvolvimento (1 semana)

### 2.1 Configuração do Ambiente Virtual

```bash
# Criar ambiente virtual
python -m venv venv_contabilidade

# Ativar ambiente (Linux/Mac)
source venv_contabilidade/bin/activate

# Ativar ambiente (Windows)
venv_contabilidade\Scripts\activate

# Instalar dependências básicas
pip install django
pip install psycopg2-binary
pip install reportlab
pip install pandas
pip install openpyxl
pip install python-decouple
```

### 2.2 Estrutura Inicial do Projeto

```bash
# Criar projeto Django
django-admin startproject contabilidade_system
cd contabilidade_system

# Criar apps principais
python manage.py startapp cadastros
python manage.py startapp lancamentos
python manage.py startapp relatorios
python manage.py startapp conciliacao
```

---

## Fase 3: Desenvolvimento por Módulos

### 3.1 Estrutura Final do Projeto

```
contabilidade_system/
├── manage.py
├── contabilidade_system/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── cadastros/
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   └── templates/cadastros/
├── lancamentos/
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   └── templates/lancamentos/
├── relatorios/
│   ├── models.py
│   ├── views.py
│   ├── utils.py
│   └── templates/relatorios/
├── conciliacao/
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   └── templates/conciliacao/
├── static/
│   ├── css/
│   ├── js/
│   └── images/
└── templates/
    └── base.html
```

### 3.2 Modelos de Dados Principais

```python
# cadastros/models.py
from django.db import models

class Empresa(models.Model):
    razao_social = models.CharField(max_length=200)
    cnpj = models.CharField(max_length=18, unique=True)
    telefone = models.CharField(max_length=15)
    email = models.EmailField()
    endereco = models.TextField()
    
    def __str__(self):
        return self.razao_social

class PlanoContas(models.Model):
    TIPO_CONTA_CHOICES = [
        ('A', 'Ativo'),
        ('P', 'Passivo'),
        ('R', 'Receita'),
        ('D', 'Despesa'),
        ('PL', 'Patrimônio Líquido'),
    ]
    
    codigo = models.CharField(max_length=10)
    descricao = models.CharField(max_length=100)
    tipo = models.CharField(max_length=2, choices=TIPO_CONTA_CHOICES)
    conta_pai = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True)
    
    class Meta:
        verbose_name_plural = "Plano de Contas"
    
    def __str__(self):
        return f"{self.codigo} - {self.descricao}"

class Cliente(models.Model):
    TIPO_PESSOA_CHOICES = [
        ('F', 'Física'),
        ('J', 'Jurídica'),
    ]
    
    nome_razao_social = models.CharField(max_length=200)
    tipo_pessoa = models.CharField(max_length=1, choices=TIPO_PESSOA_CHOICES)
    cpf_cnpj = models.CharField(max_length=18, unique=True)
    telefone = models.CharField(max_length=15)
    email = models.EmailField()
    endereco = models.TextField()
    data_cadastro = models.DateField(auto_now_add=True)
    
    def __str__(self):
        return self.nome_razao_social
```

```python
# lancamentos/models.py
from django.db import models
from cadastros.models import PlanoContas

class LancamentoContabil(models.Model):
    data = models.DateField()
    descricao = models.CharField(max_length=200)
    documento = models.CharField(max_length=50, blank=True)
    valor_total = models.DecimalField(max_digits=15, decimal_places=2)
    
    class Meta:
        verbose_name_plural = "Lançamentos Contábeis"
        ordering = ['-data', '-id']
    
    def __str__(self):
        return f"{self.data} - {self.descricao}"

class ItemLancamento(models.Model):
    TIPO_CHOICES = [
        ('D', 'Débito'),
        ('C', 'Crédito'),
    ]
    
    lancamento = models.ForeignKey(LancamentoContabil, on_delete=models.CASCADE, related_name='itens')
    conta = models.ForeignKey(PlanoContas, on_delete=models.PROTECT)
    tipo = models.CharField(max_length=1, choices=TIPO_CHOICES)
    valor = models.DecimalField(max_digits=15, decimal_places=2)
    
    def __str__(self):
        return f"{self.conta.codigo} - {self.tipo} - R$ {self.valor}"
```

---

## Fase 4: Módulos Principais

### 4.1 Módulo de Cadastros (2-3 semanas)

**Funcionalidades:**
- CRUD de empresas
- CRUD de clientes/fornecedores
- Gestão do plano de contas
- Cadastro de centros de custo
- Usuários e permisões

### 4.2 Módulo de Lançamentos (3-4 semanas)

```python
# lancamentos/forms.py
from django import forms
from .models import LancamentoContabil, ItemLancamento

class LancamentoForm(forms.ModelForm):
    class Meta:
        model = LancamentoContabil
        fields = ['data', 'descricao', 'documento']
        widgets = {
            'data': forms.DateInput(attrs={'type': 'date', 'class': 'form-control'}),
            'descricao': forms.TextInput(attrs={'class': 'form-control'}),
            'documento': forms.TextInput(attrs={'class': 'form-control'}),
        }

class ItemLancamentoForm(forms.ModelForm):
    class Meta:
        model = ItemLancamento
        fields = ['conta', 'tipo', 'valor']
        widgets = {
            'conta': forms.Select(attrs={'class': 'form-control'}),
            'tipo': forms.Select(attrs={'class': 'form-control'}),
            'valor': forms.NumberInput(attrs={'class': 'form-control', 'step': '0.01'}),
        }
```

### 4.3 Módulo de Relatórios (3-4 semanas)

```python
# relatorios/utils.py
from django.db.models import Sum, Q
from cadastros.models import PlanoContas
from lancamentos.models import ItemLancamento
import pandas as pd
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import A4
from io import BytesIO

def calcular_balancete(data_inicio, data_fim):
    """
    Calcula balancete contábil no período especificado
    """
    contas = PlanoContas.objects.all()
    resultados = []
    
    for conta in contas:
        debitos = ItemLancamento.objects.filter(
            conta=conta,
            tipo='D',
            lancamento__data__range=[data_inicio, data_fim]
        ).aggregate(total=Sum('valor'))['total'] or 0
        
        creditos = ItemLancamento.objects.filter(
            conta=conta,
            tipo='C',
            lancamento__data__range=[data_inicio, data_fim]
        ).aggregate(total=Sum('valor'))['total'] or 0
        
        saldo = debitos - creditos if conta.tipo in ['A', 'D'] else creditos - debitos
        
        resultados.append({
            'conta': conta.codigo,
            'descricao': conta.descricao,
            'debitos': debitos,
            'creditos': creditos,
            'saldo': saldo
        })
    
    return resultados

def gerar_pdf_balancete(resultados, data_inicio, data_fim):
    """
    Gera PDF do balancete
    """
    buffer = BytesIO()
    p = canvas.Canvas(buffer, pagesize=A4)
    
    # Cabeçalho
    p.setFont("Helvetica-Bold", 16)
    p.drawString(100, 800, "Balancete Contábil")
    p.setFont("Helvetica", 10)
    p.drawString(100, 780, f"Período: {data_inicio} a {data_fim}")
    
    # Conteúdo
    y = 750
    p.setFont("Helvetica-Bold", 10)
    p.drawString(50, y, "Conta")
    p.drawString(150, y, "Descrição")
    p.drawString(300, y, "Débitos")
    p.drawString(400, y, "Créditos")
    p.drawString(500, y, "Saldo")
    
    y -= 20
    p.setFont("Helvetica", 8)
    
    for item in resultados:
        if y < 50:  # Nova página se necessário
            p.showPage()
            y = 750
        
        p.drawString(50, y, item['conta'])
        p.drawString(150, y, item['descrição'])
        p.drawString(300, y, f"R$ {item['debitos']:,.2f}")
        p.drawString(400, y, f"R$ {item['creditos']:,.2f}")
        p.drawString(500, y, f"R$ {item['saldo']:,.2f}")
        y -= 15
    
    p.save()
    buffer.seek(0)
    return buffer
```

---

## Fase 5: Funcionalidades Avançadas

### 5.1 Apuração de Impostos

```python
# impostos/utils.py
def calcular_irpj_csll(lucro_contabil, data_apuracao):
    """
    Calcula IRPJ e CSLL baseado no lucro contábil
    """
    # Alíquotas (exemplo - verificar legislação atual)
    ALIQUOTA_IRPJ = 0.15
    ALIQUOTA_CSLL = 0.09
    
    irpj = lucro_contabil * ALIQUOTA_IRPJ
    csll = lucro_contabil * ALIQUOTA_CSLL
    
    return {
        'irpj': irpj,
        'csll': csll,
        'total_impostos': irpj + csll
    }
```

### 5.2 Conciliação Bancária

```python
# conciliacao/models.py
from django.db import models
from cadastros.models import Empresa

class ContaBancaria(models.Model):
    empresa = models.ForeignKey(Empresa, on_delete=models.CASCADE)
    banco = models.CharField(max_length=50)
    agencia = models.CharField(max_length=10)
    conta = models.CharField(max_length=15)
    saldo_atual = models.DecimalField(max_digits=15, decimal_places=2, default=0)
    
    def __str__(self):
        return f"{self.banco} - Ag {self.agencia} C/C {self.conta}"

class ExtratoBancario(models.Model):
    conta = models.ForeignKey(ContaBancaria, on_delete=models.CASCADE)
    data = models.DateField()
    descricao = models.CharField(max_length=200)
    documento = models.CharField(max_length=50)
    valor = models.DecimalField(max_digits=15, decimal_places=2)
    tipo = models.CharField(max_length=1, choices=[('C', 'Crédito'), ('D', 'Débito')])
    conciliado = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-data', '-id']
```

---

## Fase 6: Testes e Validação (2 semanas)

### 6.1 Testes Unitários

```python
# lancamentos/tests.py
from django.test import TestCase
from django.db.models import Sum
from .models import LancamentoContabil, ItemLancamento
from cadastros.models import PlanoContas

class LancamentoTestCase(TestCase):
    def setUp(self):
        self.conta_caixa = PlanoContas.objects.create(
            codigo="1.1.01",
            descricao="Caixa",
            tipo="A"
        )
        self.conta_receita = PlanoContas.objects.create(
            codigo="3.1.01",
            descricao="Receita de Vendas",
            tipo="R"
        )
    
    def test_partidas_dobradas(self):
        """Testa se débitos igualam créditos"""
        lancamento = LancamentoContabil.objects.create(
            data="2024-01-15",
            descricao="Venda à vista",
            valor_total=1000.00
        )
        
        ItemLancamento.objects.create(
            lancamento=lancamento,
            conta=self.conta_caixa,
            tipo="D",
            valor=1000.00
        )
        
        ItemLancamento.objects.create(
            lancamento=lancamento,
            conta=self.conta_receita,
            tipo="C",
            valor=1000.00
        )
        
        total_debitos = ItemLancamento.objects.filter(
            lancamento=lancamento, tipo='D'
        ).aggregate(total=Sum('valor'))['total']
        
        total_creditos = ItemLancamento.objects.filter(
            lancamento=lancamento, tipo='C'
        ).aggregate(total=Sum('valor'))['total']
        
        self.assertEqual(total_debitos, total_creditos)
        self.assertEqual(total_debitos, 1000.00)
```

---

## Fase 7: Implantação e Documentação

### 7.1 Configuração de Deploy

**settings.py para produção:**
```python
import os
from decouple import config

DEBUG = config('DEBUG', default=False, cast=bool)
SECRET_KEY = config('SECRET_KEY')
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost').split(',')

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST'),
        'PORT': config('DB_PORT', default='5432'),
    }
}

# Configurações de arquivos estáticos
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

### 7.2 Arquivo requirements.txt
```
Django==4.2.7
psycopg2-binary==2.9.7
reportlab==4.0.4
pandas==2.1.3
openpyxl==3.1.2
python-decouple==3.8
whitenoise==6.6.0
gunicorn==21.2.0
```

---

## Stack Tecnológica Completa

| Componente | Tecnologia | Status |
|------------|------------|---------|
| Backend Framework | Django | ✅ Gratuito |
| Database | PostgreSQL | ✅ Gratuito |
| Frontend | Bootstrap 5 | ✅ Gratuito |
| Relatórios PDF | ReportLab | ✅ Gratuito |
| Planilhas | Pandas + OpenPyXL | ✅ Gratuito |
| Gráficos | Chart.js | ✅ Gratuito |
| Deploy | Heroku/Railway | ✅ Gratuito |
| Versionamento | Git + GitHub | ✅ Gratuito |

---

## Cronograma Estimado

| Fase | Duração | Entregáveis |
|------|---------|-------------|
| 1. Planejamento | 2 semanas | Documento de requisitos, arquitetura |
| 2. Ambiente | 1 semana | Ambiente configurado, projeto Django |
| 3. Cadastros | 2 semanas | CRUD empresas, clientes, plano contas |
| 4. Lançamentos | 3 semanas | Sistema completo de lançamentos |
| 5. Relatórios | 3 semanas | Balancete, DRE, Balanço em PDF/Excel |
| 6. Impostos | 2 semanas | Apuração IRPJ, CSLL, PIS, COFINS |
| 7. Conciliação | 2 semanas | Import extrato, conciliação |
| 8. Testes | 2 semanas | Testes unitários, validação |
| 9. Deploy | 1 semana | Sistema em produção |
| **Total** | **18 semanas** | **Sistema completo** |

---

## Dicas Importantes

### 🔒 Segurança
1. **Validação de dados**: Sempre validar entradas do usuário
2. **Autenticação**: Sistema de login seguro
3. **Permissões**: Controle de acesso por usuário
4. **Backup**: Rotina automática de backup
5. **HTTPS**: Certificado SSL em produção

### 💾 Performance
1. **Indexação**: Índices adequados no banco de dados
2. **Paginação**: Listas grandes com paginação
3. **Cache**: Cache de relatórios complexos
4. **Otimização**: Queries otimizadas

### 📊 Qualidade
1. **Testes**: Cobertura mínima de 80% de testes
2. **Documentação**: Manual do usuário e técnico
3. **Logs**: Sistema de logging para auditoria
4. **Validações**: Regras contábeis validadas automaticamente

### 🚀 Próximos Passos Imediatos
1. Configurar ambiente virtual
2. Criar projeto Django base
3. Implementar modelos de dados
4. Criar interface de cadastros básicos

---

**📝 Nota:** Este roteiro é um guia completo, mas deve ser adaptado conforme as necessidades específicas do seu projeto. Recomendo começar com as funcionalidades básicas e evoluir gradualmente.

Salve este arquivo como `roteiro_software_contabilidade.md` para referência futura!
