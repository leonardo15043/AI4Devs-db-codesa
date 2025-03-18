```Leonardo : ``` 
Eres un experto en bases de datos y Prisma ORM. Tu misión es convertir el siguiente diagrama entidad-relación (ERD) en un script SQL para PostgreSQL y expandir la estructura con migraciones de Prisma.

## Tareas:
1. **Convertir el ERD a SQL**
   - Genera un script SQL basado en el ERD proporcionado.
   - Asegúrate de definir claves primarias, foráneas e índices adecuados.
   - Aplica normalización para mejorar la eficiencia de la base de datos.

2. **Expansión con Prisma ORM**
   - Analiza la estructura de la base de datos existente.
   - Implementa migraciones en Prisma para reflejar cualquier cambio necesario.
   - Usa Prisma Migrate para aplicar y verificar los cambios.

3. **Buenas Prácticas**
   - Define índices en campos clave para optimizar las consultas.
   - Normaliza la base de datos para evitar redundancias.
   - Utiliza nombres de tablas y columnas descriptivos y coherentes.

4. **Validación y Pruebas**
   - Conéctate a PostgreSQL con PGAdmin u otra herramienta similar.
   - Verifica la correcta creación de la estructura.
   - Inserta datos de prueba y ejecuta consultas para comprobar la integridad.

## ERD en formato Mermaid:

```
mermaid
erDiagram
     COMPANY {
         int id PK
         string name
     }
     EMPLOYEE {
         int id PK
         int company_id FK
         string name
         string email
         string role
         boolean is_active
     }
     POSITION {
         int id PK
         int company_id FK
         int interview_flow_id FK
         string title
         text description
         string status
         boolean is_visible
         string location
         text job_description
         text requirements
         text responsibilities
         numeric salary_min
         numeric salary_max
         string employment_type
         text benefits
         text company_description
         date application_deadline
         string contact_info
     }
     INTERVIEW_FLOW {
         int id PK
         string description
     }
     INTERVIEW_STEP {
         int id PK
         int interview_flow_id FK
         int interview_type_id FK
         string name
         int order_index
     }
     INTERVIEW_TYPE {
         int id PK
         string name
         text description
     }
     CANDIDATE {
         int id PK
         string firstName
         string lastName
         string email
         string phone
         string address
     }
     APPLICATION {
         int id PK
         int position_id FK
         int candidate_id FK
         date application_date
         string status
         text notes
     }
     INTERVIEW {
         int id PK
         int application_id FK
         int interview_step_id FK
         int employee_id FK
         date interview_date
         string result
         int score
         text notes
     }

     COMPANY ||--o{ EMPLOYEE : employs
     COMPANY ||--o{ POSITION : offers
     POSITION ||--|| INTERVIEW_FLOW : assigns
     INTERVIEW_FLOW ||--o{ INTERVIEW_STEP : contains
     INTERVIEW_STEP ||--|| INTERVIEW_TYPE : uses
     POSITION ||--o{ APPLICATION : receives
     CANDIDATE ||--o{ APPLICATION : submits
     APPLICATION ||--o{ INTERVIEW : has
     INTERVIEW ||--|| INTERVIEW_STEP : consists_of
     EMPLOYEE ||--o{ INTERVIEW : conducts

```

```ChatGPT : ``` 

Voy a generar el script SQL para PostgreSQL basado en el ERD proporcionado y luego lo adaptaré a Prisma ORM con migraciones. Comenzaré con el script SQL.

```
CREATE TABLE company (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

CREATE TABLE employee (
    id SERIAL PRIMARY KEY,
    company_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(100) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    FOREIGN KEY (company_id) REFERENCES company(id) ON DELETE CASCADE
);

CREATE TABLE interview_flow (
    id SERIAL PRIMARY KEY,
    description TEXT NOT NULL
);

CREATE TABLE interview_type (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT
);

CREATE TABLE interview_step (
    id SERIAL PRIMARY KEY,
    interview_flow_id INT NOT NULL,
    interview_type_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    order_index INT NOT NULL,
    FOREIGN KEY (interview_flow_id) REFERENCES interview_flow(id) ON DELETE CASCADE,
    FOREIGN KEY (interview_type_id) REFERENCES interview_type(id) ON DELETE CASCADE
);

CREATE TABLE position (
    id SERIAL PRIMARY KEY,
    company_id INT NOT NULL,
    interview_flow_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) NOT NULL,
    is_visible BOOLEAN NOT NULL DEFAULT TRUE,
    location VARCHAR(255),
    job_description TEXT,
    requirements TEXT,
    responsibilities TEXT,
    salary_min NUMERIC,
    salary_max NUMERIC,
    employment_type VARCHAR(100),
    benefits TEXT,
    company_description TEXT,
    application_deadline DATE,
    contact_info VARCHAR(255),
    FOREIGN KEY (company_id) REFERENCES company(id) ON DELETE CASCADE,
    FOREIGN KEY (interview_flow_id) REFERENCES interview_flow(id) ON DELETE SET NULL
);

CREATE TABLE candidate (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    address TEXT
);

CREATE TABLE application (
    id SERIAL PRIMARY KEY,
    position_id INT NOT NULL,
    candidate_id INT NOT NULL,
    application_date DATE NOT NULL DEFAULT CURRENT_DATE,
    status VARCHAR(50) NOT NULL,
    notes TEXT,
    FOREIGN KEY (position_id) REFERENCES position(id) ON DELETE CASCADE,
    FOREIGN KEY (candidate_id) REFERENCES candidate(id) ON DELETE CASCADE
);

CREATE TABLE interview (
    id SERIAL PRIMARY KEY,
    application_id INT NOT NULL,
    interview_step_id INT NOT NULL,
    employee_id INT NOT NULL,
    interview_date DATE NOT NULL,
    result VARCHAR(100),
    score INT,
    notes TEXT,
    FOREIGN KEY (application_id) REFERENCES application(id) ON DELETE CASCADE,
    FOREIGN KEY (interview_step_id) REFERENCES interview_step(id) ON DELETE CASCADE,
    FOREIGN KEY (employee_id) REFERENCES employee(id) ON DELETE SET NULL
);


```

Ahora procederé a convertir esta estructura en un esquema de Prisma ORM y a definir las migraciones necesarias. ¿Quieres que lo haga en un nuevo archivo o continúe en este?

```Leonardo : ``` 

continua con el esquema de prisma pero en un archivo nuevo 

```ChatGPT : ```

```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Company {
  id       Int      @id @default(autoincrement())
  name     String   @db.VarChar(255)
  employees Employee[]
  positions Position[]
}

model Employee {
  id        Int      @id @default(autoincrement())
  company   Company  @relation(fields: [companyId], references: [id])
  companyId Int
  name      String   @db.VarChar(255)
  email     String   @unique @db.VarChar(255)
  role      String   @db.VarChar(100)
  isActive  Boolean  @default(true)
  interviews Interview[]
}

model InterviewFlow {
  id       Int             @id @default(autoincrement())
  description String       
  positions Position[]
  steps    InterviewStep[]
}

model InterviewStep {
  id                Int           @id @default(autoincrement())
  interviewFlow     InterviewFlow @relation(fields: [interviewFlowId], references: [id])
  interviewFlowId   Int
  interviewType     InterviewType @relation(fields: [interviewTypeId], references: [id])
  interviewTypeId   Int
  name             String @db.VarChar(255)
  orderIndex       Int
  interviews       Interview[]
}

model InterviewType {
  id          Int     @id @default(autoincrement())
  name        String  @db.VarChar(255)
  description String?
  steps       InterviewStep[]
}

model Position {
  id                 Int             @id @default(autoincrement())
  company            Company         @relation(fields: [companyId], references: [id])
  companyId          Int
  interviewFlow      InterviewFlow   @relation(fields: [interviewFlowId], references: [id])
  interviewFlowId    Int?
  title              String          @db.VarChar(255)
  description        String?
  status             String          @db.VarChar(50)
  isVisible          Boolean         @default(true)
  location           String?
  jobDescription     String?
  requirements       String?
  responsibilities   String?
  salaryMin          Float?
  salaryMax          Float?
  employmentType     String?
  benefits           String?
  companyDescription String?
  applicationDeadline Date?
  contactInfo        String?
  applications       Application[]
}

model Candidate {
  id        Int      @id @default(autoincrement())
  firstName String   @db.VarChar(255)
  lastName  String   @db.VarChar(255)
  email     String   @unique @db.VarChar(255)
  phone     String?  @db.VarChar(20)
  address   String?
  applications Application[]
}

model Application {
  id            Int         @id @default(autoincrement())
  position      Position    @relation(fields: [positionId], references: [id])
  positionId    Int
  candidate     Candidate   @relation(fields: [candidateId], references: [id])
  candidateId   Int
  applicationDate Date      @default(now())
  status        String      @db.VarChar(50)
  notes         String?
  interviews    Interview[]
}

model Interview {
  id              Int             @id @default(autoincrement())
  application     Application     @relation(fields: [applicationId], references: [id])
  applicationId   Int
  interviewStep   InterviewStep   @relation(fields: [interviewStepId], references: [id])
  interviewStepId Int
  employee        Employee?       @relation(fields: [employeeId], references: [id])
  employeeId      Int?
  interviewDate   Date
  result          String?         @db.VarChar(100)
  score          Int?
  notes          String?
}

```

El esquema de Prisma ha sido generado con todas las relaciones y restricciones basadas en el ERD. Ahora puedes ejecutar prisma migrate dev --name init para aplicar la migración a tu base de datos PostgreSQL. ¿Necesitas ajustes o agregar más validaciones?