# Lộ trình học IT từ cơ bản đến nâng cao

> Mục tiêu: học theo tầng kiến thức, không học rời rạc từng công nghệ. Nên hoàn thành nền tảng trước khi leo lên tầng cao hơn.

---

## 1. TỔNG QUAN CẤU TRÚC

```text
LEVEL 0
Máy tính & công cụ
        ↓
LEVEL 1
Programming Fundamentals
        ↓
LEVEL 2
Git + Linux + Database
        ↓
LEVEL 3
Web / Application Development
        ↓
LEVEL 4
Backend / Frontend / Mobile
        ↓
LEVEL 5
Software Engineering
        ↓
LEVEL 6
DevOps + Cloud
        ↓
LEVEL 7
System Design + Distributed Systems
        ↓
LEVEL 8
Security
        ↓
LEVEL 9
AI / Data / Machine Learning
        ↓
LEVEL 10
Advanced Architecture / AI Engineering
```

---

## 2. LEVEL 0 — COMPUTER FUNDAMENTALS

Đây là nền móng để hiểu cách máy tính chạy chương trình.

### Cần biết
- CPU
- RAM
- SSD/HDD
- GPU
- Process
- Thread
- File system
- OS

### Hệ điều hành
- Windows
- Linux
- macOS

### Command line

#### Windows
```bash
cd
dir
mkdir
copy
move
del
```

#### Linux
```bash
cd
ls
mkdir
cp
mv
rm
cat
grep
chmod
```

### Quan trọng cần hiểu
```text
Program
   ↓
Process
   ↓
Thread
   ↓
CPU
   ↓
Memory
```

Không cần học sâu phần cứng, nhưng phải hiểu cách chương trình được xử lý bởi máy tính.

---

## 3. LEVEL 1 — PROGRAMMING FUNDAMENTALS

Đây là level quan trọng nhất đối với người mới bắt đầu.

### Ngôn ngữ nên chọn

#### Backend
- Python
- Java
- C#
- Node.js / TypeScript

#### Frontend
- JavaScript
- TypeScript

#### Low-level
- C
- C++
- Rust

> Nếu mục tiêu là Web + AI, combo tốt nhất là: Python + JavaScript/TypeScript

### Kiến thức bắt buộc
#### Syntax
- variable
- datatype
- operator
- if/else
- loop
- function

#### Data structures
- Array / List
- Stack
- Queue
- Set
- Map / Dictionary
- Tree
- Graph

#### Algorithms
- Searching
- Sorting
- Recursion
- Two pointers
- Binary search
- Graph traversal
- Dynamic programming

#### OOP
- Class
- Object
- Inheritance
- Encapsulation
- Polymorphism
- Abstraction

#### Error handling
- try / catch
- throw
- exception

---

## 4. LEVEL 2 — GIT + LINUX + DATABASE

### Git
Phải biết các lệnh cơ bản:

```bash
git init
git clone
git status
git add
git commit
git push
git pull
git fetch
git branch
git checkout
git switch
git merge
git rebase
git stash
git reset
git revert
```

Hiểu flow:
```text
Working Directory
        ↓
Staging Area
        ↓
Commit
        ↓
Remote
```

### GitHub
- Repository
- Branch
- Pull Request
- Issue
- Merge
- Release
- README
- .gitignore

### Linux
Nên dùng thành thạo:
```bash
ssh
systemctl
ps
top
grep
curl
wget
chmod
chown
```

### Database
- SQL cơ bản
- Relational model
- Transactions
- Index
- Normalization

---

## 5. LEVEL 3 — WEB / APPLICATION DEVELOPMENT

Ở giai đoạn này, bạn bắt đầu xây dựng ứng dụng thực tế.

### Frontend cơ bản
- HTML
- CSS
- JavaScript

### Sau đó học
- TypeScript
- React / Vue / Angular

> Nếu đi theo hướng phổ biến: React + TypeScript

### Backend
- Java + Spring Boot
- Python + Flask/FastAPI/Django
- Node.js + Express/NestJS

### Mục tiêu
- Xây dựng web app từ front-end đến back-end
- Nắm được request/response
- Kết nối database
- Tương tác với API

---

## 6. LEVEL 4 — BACKEND / FRONTEND / MOBILE

Tùy theo định hướng, bạn có thể đi vào một trong các lĩnh vực sau:

### Backend Engineer
- Java / Spring Boot
- Python / FastAPI / Django
- Node.js / NestJS

### Frontend Engineer
- TypeScript
- React
- Next.js
- State management
- UI/UX basics

### Mobile Engineer
- Android
- iOS
- Flutter
- React Native

> Nên chọn 1 stack chính và đào sâu nó, thay vì học quá nhiều framework cùng lúc.

---

## 7. LEVEL 5 — SOFTWARE ENGINEERING

Đây là phần phân biệt giữa:
- biết code
- biết phát triển phần mềm

### Clean Code
- Naming
- Function design
- Modularity
- Separation of concerns
- DRY
- SOLID

### Design Patterns
- Factory
- Strategy
- Observer
- Adapter
- Decorator
- Repository
- Dependency Injection

### Architecture
- Monolith
- Layered Architecture
- MVC
- Clean Architecture
- Hexagonal Architecture
- Microservices

### Testing
- Unit test
- Integration test
- E2E test
- Mock
- Test coverage

### Công cụ test phổ biến
- JUnit
- Pytest
- Jest
- Playwright
- Selenium

---

## 8. LEVEL 6 — DATABASE & SQL

### SQL bắt buộc
```sql
SELECT
INSERT
UPDATE
DELETE
JOIN
GROUP BY
ORDER BY
HAVING
```

### Concepts
- Primary Key
- Foreign Key
- Index
- Constraint
- Transaction
- Normalization
- Relationship

### Database phổ biến
- MySQL
- PostgreSQL
- SQL Server
- Oracle

> Nên học sâu 1 database trước, ví dụ PostgreSQL

### NoSQL
- MongoDB
- Redis
- Elasticsearch

---

## 9. LEVEL 7 — NETWORKING

Đây là phần rất quan trọng với người làm web và backend.

### Cần hiểu
- IP
- Port
- DNS
- HTTP
- HTTPS
- TCP
- UDP
- Socket
- Request
- Response
- Cookie
- Session
- JWT

### Flow cơ bản
```text
Browser
   ↓
DNS
   ↓
Server IP
   ↓
TCP
   ↓
HTTP
   ↓
Backend
   ↓
Database
```

---

## 10. LEVEL 8 — API

### REST API
Phải biết:
```http
GET /users
POST /users
GET /users/123
PUT /users/123
DELETE /users/123
```

### Khái niệm cần có
- Authentication
- Authorization
- Pagination
- Filtering
- Sorting
- Validation
- Error handling
- Rate limiting

### API technologies
- REST
- GraphQL
- WebSocket
- gRPC

> Không cần học tất cả cùng lúc; học REST trước là quan trọng nhất.

---

## 11. LEVEL 9 — SECURITY

Developer càng lên cao càng cần hiểu security.

### Web Security
- Authentication
- Authorization
- Session Security
- Password Hashing
- CSRF
- XSS
- SQL Injection
- SSRF
- IDOR / BOLA
- CORS
- Rate Limiting

### OWASP
Nên biết: OWASP Top 10

---

## 12. LEVEL 10 — DEVOPS

Đây là giai đoạn chuyển từ developer sang engineer.

### Linux
- ssh
- systemctl
- ps
- top
- grep
- curl
- wget
- chmod
- chown

### Docker
- Image
- Container
- Dockerfile
- Volume
- Network
- Registry
- Docker Compose

### CI/CD
```text
GitHub
  ↓
Push
  ↓
GitHub Actions
  ↓
Build
  ↓
Test
  ↓
Deploy
```

---

## 13. LEVEL 11 — CLOUD

Sau khi đã hiểu Docker và CI/CD, thì nên học cloud.

### AWS
- EC2
- S3
- RDS
- Lambda
- VPC
- IAM
- CloudWatch

### Azure
- VM
- Blob Storage
- Azure SQL
- Functions
- Entra ID

### GCP
- Compute Engine
- Cloud Storage
- Cloud SQL
- Cloud Run
- BigQuery

> Không cần học cả 3; chọn 1 nền tảng để đi sâu.

---

## 14. LEVEL 12 — SYSTEM DESIGN

Đây là level rất quan trọng nếu muốn lên senior.

### Cần hiểu
- Load Balancer
- Cache
- CDN
- Message Queue
- Database Replication
- Database Sharding
- Horizontal Scaling
- Vertical Scaling
- API Gateway
- Service Discovery
- Distributed Systems

### Ví dụ kiến trúc
```text
             Users
                ↓
             CDN
                ↓
         Load Balancer
          ↙          ↘
      Server 1      Server 2
          ↓            ↓
             Cache
                ↓
           Database
```

---

## 15. LEVEL 13 — DISTRIBUTED SYSTEMS

Nâng cao hơn system design.

### Khái niệm
- Kafka
- RabbitMQ
- Redis
- Zookeeper
- Consensus
- Replication
- Partitioning
- Event-driven architecture
- Eventually consistent systems

### Vấn đề cần hiểu
- Scalability
- Availability
- Consistency
- Fault tolerance
- Latency
- Throughput

---

## 16. LEVEL 14 — DATA ENGINEERING

Nếu đi hướng data:

- Python
- SQL
- Pandas
- NumPy
- ETL
- Data Warehouse
- Data Lake
- Spark
- Kafka
- Airflow
- dbt

### Công nghệ khác
- BigQuery
- Snowflake
- Databricks

---

## 17. LEVEL 15 — AI / MACHINE LEARNING

### Toán học cơ bản
- Linear Algebra
- Probability
- Statistics
- Calculus

### Machine Learning
- Regression
- Classification
- Clustering
- Dimensionality Reduction
- Anomaly Detection

### Algorithms phổ biến
- Linear Regression
- Logistic Regression
- Decision Tree
- Random Forest
- SVM
- K-Means
- Isolation Forest

### Deep Learning
- Neural Network
- CNN
- RNN
- LSTM
- Transformer

### Framework
- PyTorch
- TensorFlow

---

## 18. LEVEL 16 — GENERATIVE AI

Sau ML/DL mới đi sâu GenAI.

### Khái niệm
- LLM
- Transformer
- Tokenization
- Embedding
- Vector Database
- RAG
- Fine-tuning
- Prompt Engineering
- Agents
- Tool Calling
- Evaluation

### Công nghệ
- OpenAI API
- Hugging Face
- LangChain
- LlamaIndex
- FAISS
- Qdrant
- Pinecone
- Milvus

---

## 19. LEVEL 17 — AI ENGINEERING

Đây là mức xây hệ thống AI thực tế, không chỉ gọi API.

```text
User
 ↓
Application
 ↓
AI Orchestrator
 ↓
LLM
 ↓
Tools
 ↓
Database
 ↓
Vector DB
 ↓
Evaluation
 ↓
Monitoring
```

### Vấn đề nâng cao
- RAG
- Agent
- Memory
- Tool Calling
- Structured Output
- Guardrails
- LLM Evaluation
- Prompt Versioning
- Model Routing
- AI Observability
- AI Security

---

## 20. LEVEL 18 — ADVANCED SOFTWARE ARCHITECTURE

Nâng cấp kiến trúc hệ thống:

- Distributed Systems
- Cloud Architecture
- Event-driven Architecture
- Microservices
- Service Mesh
- High Availability
- Fault Tolerance
- Observability
- Scalability

### Công nghệ gặp nhiều
- Kubernetes
- Terraform
- Istio
- Prometheus
- Grafana
- OpenTelemetry
- ArgoCD

---

## 21. SPECIALIZATION

Đến giai đoạn này, bạn nên chọn một hướng rõ ràng.

### Backend Engineer
- Java/Spring
- Database
- API
- Distributed Systems
- Cloud

### Frontend Engineer
- TypeScript
- React
- Next.js
- Performance
- Frontend Architecture

### DevOps / Cloud
- Linux
- Docker
- Kubernetes
- AWS
- Terraform
- CI/CD

### Cybersecurity
- Network
- Linux
- Web Security
- Pentesting
- Cloud Security

### Data Engineer
- Python
- SQL
- ETL
- Spark
- Kafka
- Cloud

### ML Engineer
- Python
- Statistics
- ML
- Deep Learning
- MLOps

### AI Engineer
- Python
- LLM
- RAG
- Agents
- Vector DB
- AI Evaluation
- AI Infrastructure

---

## 22. BẢN ĐỒ IT GỘP LẠI

```text
                 IT
                  │
     ┌────────────┼────────────┐
     ↓            ↓            ↓
 Programming    Computer      Network
     │            │            │
     └────────────┼────────────┘
                  ↓
             Development
          ↙       ↓        ↘
      Frontend  Backend   Mobile
          │       │
          └───┬───┘
              ↓
      Software Engineering
              ↓
       DevOps / Cloud
              ↓
       System Architecture
              ↓
      ┌───────┼────────┐
      ↓       ↓        ↓
   Security  Data      AI
                       ↓
                 AI Engineering
```

---

## 23. KHÔNG NÊN HỌC QUÁ NHIỀU CÔNG NGHỆ CÙNG LÚC

Nếu bạn đã có nền tảng như:
- Python
- Flask
- Java
- Spring Boot
- NestJS
- SQL
- Git/GitHub
- Machine Learning
- Isolation Forest
- Web Security

thì vấn đề không phải là thiếu công nghệ, mà là thiếu chiều sâu kiến thức.

### Giai đoạn nên ưu tiên
```text
① Programming Fundamentals
        ↓
② OOP + Data Structures + Algorithms
        ↓
③ Git + Linux + Networking
        ↓
④ SQL + Database
        ↓
⑤ Backend thật chắc
        ↓
⑥ REST API + Authentication
        ↓
⑦ Testing + Clean Code + SOLID
        ↓
⑧ Docker + CI/CD
        ↓
⑨ System Design
        ↓
⑩ AI / ML / LLM
```

> Chưa cần lao vào Kubernetes, Microservices, Kafka, Terraform khi ①–⑦ chưa chắc.

---

## 24. ĐƯỜNG ĐI HỢP LÝ CHO MỤC TIÊU BACKEND + AI ENGINEER

```text
Java / Spring Boot
        ↓
Backend Engineering
        ↓
System Design
        ↓
Docker / Cloud
        ↓
Python / ML
        ↓
LLM / RAG / AI Engineering
```

---

## 25. KHUYẾN NGHỊ THỰC TẾ

### Nếu bạn mới bắt đầu
1. Học Python hoặc JavaScript
2. Nắm chắc lập trình cơ bản
3. Học Git + Linux
4. Học SQL và database
5. Học backend hoặc frontend
6. Học API + authentication
7. Học clean code + testing
8. Học Docker + CI/CD
9. Học cơ bản system design
10. Học AI/LLM khi nền tảng đã chắc

### Nếu bạn đã có nền tảng nhưng chưa sâu
- Tập trung vào OOP + DSA + SQL + API
- Làm dự án thực tế 1-2 dự án
- Viết code sạch + test
- Tạo portfolio bằng project thật
- Bắt đầu mở rộng sang cloud, system design, AI

---

## 26. KẾT LUẬN

IT không phải là học tất cả công nghệ trong cùng một lúc. Nền tảng mới là điều quan trọng nhất.

Cách học đúng là:
- học kiến thức từ thấp đến cao
- hiểu rõ từng tầng
- thực hành project
- lặp lại và đào sâu từng phần
- chọn một hướng rõ ràng

Nếu bạn muốn học hiệu quả, hãy đi từng bước theo thứ tự sau:

```text
Programming Fundamentals
→ Git + Linux
→ SQL + Database
→ Backend / Frontend
→ REST API
→ Clean Code + Testing
→ Docker + CI/CD
→ System Design
→ AI / LLM
```

---

Bạn có thể dùng file này như roadmap chính để học từng level một, hoặc chỉnh sửa lại theo hướng công việc bạn muốn tập trung: Backend, Frontend, DevOps, Data, AI Engineer.
