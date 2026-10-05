# 🗃️ Object Storage com Garage

Implementação de arquitetura para upload e armazenamento de arquivos utilizando Object Storage S3-compatible com Garage.

---

## 📌 Visão Geral

Atualmente, arquivos eram armazenados como base64 no banco de dados, o que gera:

- aumento significativo de armazenamento
- piora de performance
- backups mais pesados
- maior acoplamento entre dados e arquivos

A nova abordagem separa responsabilidades utilizando Object Storage.

---

## 🏗️ Arquitetura
Cliente → API → Garage (Object Storage)
PostgreSQL (metadados)


- **Garage**: responsável pelo armazenamento dos arquivos  
- **PostgreSQL**: responsável apenas pelos metadados  

---

## 📦 Sobre o Garage

O Garage é um object storage:

- S3-compatible  
- distribuído  
- open source  
- leve e self-hosted  

### 🔁 Alternativas similares

- Amazon S3
- MinIO  
- Ceph  

---

## 🧠 Conceito de S3

O Amazon S3 define uma **API HTTP (REST)** amplamente adotada como padrão de mercado para armazenamento de objetos.

Não se trata de um protocolo formal como HTTP ou TCP, mas sim de uma **especificação de API de facto**, baseada em:

- endpoints HTTP  
- convenções de requisição/resposta  
- autenticação (AWS Signature V4)  

O Garage implementa essa API, garantindo compatibilidade com ferramentas padrão.

---

## ☕ Integração com Java

A integração pode ser feita utilizando o AWS SDK for Java.

O SDK é responsável por:

- montar requisições HTTP  
- assinar requests (Signature V4)  
- gerenciar retries  
- serializar/deserializar dados  

### 🔌 Fluxo de comunicação
```
API (Java)
↓
AWS SDK
↓
HTTP (REST)
↓
Garage (S3-compatible)
```


---

## 🗄️ Modelo de Dados

Arquivos não são mais armazenados no banco.

Apenas **metadados + referência**:

```json
{
  "id": "uuid",
  "file_name": "documento.pdf",
  "bucket": "meu-bucket",
  "object_key": "user-123/abc123.pdf",
  "content_type": "application/pdf",
  "size": 245678,
  "created_at": "..."
}
```

👉 os arquivos ficam armazenados nos discos do(s) servidor(es) onde o Garage está rodando, mas não como um arquivo normal, diferente de um /uploads/file.pdf, o Garage:

👉 quebra e organiza os dados internamente como:

- blobs (chunks de dados)
- hashing (identificação por conteúdo)
- distribuição entre nós (se cluster)

- exemplo: /data/garage/blobs/ab/cd/ef123456...

### Garage com Docker 🛅:
volumes:
  - ./garage-data:/var/lib/garage

os arquivos vão ficar em:
 - ./garage-data

#### 🔗  Container do Garage: https://github.com/d0uglas77p/object-storage-container
