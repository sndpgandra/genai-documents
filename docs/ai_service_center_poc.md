# AI Service Center POC - Technical 1-Pager

## Executive Summary
Transform traditional UI-driven HR Benefits service center into a conversational AI agent that handles employee benefit inquiries through natural language chat, maintaining same capabilities with improved user experience and efficiency.

## Business Value
- **Reduced Training Time**: Natural language interface eliminates need for complex UI training
- **Improved Accuracy**: AI agent ensures consistent responses and reduces human error
- **Enhanced Compliance**: Built-in safeguards for HIPAA/PI data handling
- **Scalable Solution**: Foundation for expanding to other service center functions

## Technical Architecture

### System Architecture
```mermaid
graph TB
    subgraph "Frontend Layer"
        A[Streamlit Chat UI]
        A1[Chat Interface]
        A2[Session Management]
        A3[Message History]
        A --> A1
        A --> A2
        A --> A3
    end
    
    subgraph "Backend Layer"
        B[FastAPI Backend]
        B1[Chat Endpoints]
        B2[Session Handler]
        B3[AI Agent Controller]
        B4[API Gateway]
        B --> B1
        B --> B2
        B --> B3
        B --> B4
    end
    
    subgraph "AI Layer"
        C[Local LLM - Llama 3.1 8B]
        C1[Ollama Server]
        C2[Prompt Engineering]
        C3[Response Generation]
        C --> C1
        C --> C2
        C --> C3
    end
    
    subgraph "Data Layer"
        D[Mock APIs]
        D1[Employee Data Service]
        D2[Benefits Data Service]
        D3[Eligibility Rules Engine]
        D4[JSON Data Store]
        D --> D1
        D --> D2
        D --> D3
        D --> D4
    end
    
    A1 -->|HTTP Requests| B1
    B2 -->|Store/Retrieve| A2
    B3 -->|LLM Queries| C2
    B4 -->|Data Requests| D1
    B4 -->|Benefits Queries| D2
    B4 -->|Eligibility Checks| D3
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style C fill:#e8f5e8
    style D fill:#fff3e0
```

### Technology Stack
- **Frontend**: Streamlit (Python-based chat interface)
- **Backend**: FastAPI (REST API layer)
- **LLM**: Llama 3.1 8B via Ollama (local deployment)
- **Data**: JSON files for mocked employee/benefits data
- **Session Management**: In-memory storage for chat history

## Implementation Approach

### Phase 1: Core MVP (Days 1-3)
1. **Environment Setup**
   - Install Ollama and Llama 3.1 8B model
   - Create FastAPI backend with basic endpoints
   - Build Streamlit chat interface

2. **Mock Data Creation**
   - Employee profiles with personal info
   - 401k and Medical plan data
   - Eligibility rules implementation

3. **AI Agent Development**
   - Natural language processing for employee identification
   - Benefits query handling
   - Update request processing

### Phase 2: Demo Scenarios (Days 4-5)
1. **Happy Path Scenarios**
   - 401k contribution rate inquiry and increase
   - Medical plan details and dependent addition

2. **Error Handling**
   - Invalid employee information
   - Eligibility restrictions
   - API failure graceful handling

## Demo Scenarios

### Scenario 1: 401k Inquiry (Happy Path)
**User**: "Hi, I'm John Smith, employee ID 12345. I want to check my 401k contribution rate and maybe increase it."
**Expected Flow**: Verify identity → Retrieve 401k info → Present current rate → Process increase request

### Scenario 2: Medical Plan Update (Happy Path)
**User**: "I need to add my newborn to my medical plan. My name is Sarah Johnson, DOB 01/15/1985."
**Expected Flow**: Verify identity → Check medical plan → Validate dependent eligibility → Process addition

### Scenario 3: Eligibility Error
**User**: "I'm a part-time employee, why can't I enroll in medical?"
**Expected Flow**: Verify identity → Check eligibility → Explain requirements (1000 hrs or 1 year rule)

### Data Flow Diagram
```mermaid
sequenceDiagram
    participant U as Service Rep
    participant UI as Streamlit UI
    participant API as FastAPI Backend
    participant LLM as Llama 3.1 8B
    participant DS as Data Service
    participant DB as JSON Store
    
    U->>UI: Enter employee inquiry
    UI->>API: Send chat message
    API->>LLM: Extract employee info + intent
    LLM->>API: Structured data (ID, request type)
    API->>DS: Query employee data
    DS->>DB: Fetch employee record
    DB->>DS: Return employee data
    DS->>API: Employee + benefits info
    API->>DS: Check eligibility rules
    DS->>API: Eligibility status
    API->>LLM: Generate response with data
    LLM->>API: Natural language response
    API->>UI: Formatted chat response
    UI->>U: Display conversation
    
    Note over U,DB: Happy Path: Successful Query
    
    rect rgb(255, 200, 200)
        U->>UI: Invalid employee ID
        UI->>API: Send chat message
        API->>LLM: Extract employee info
        LLM->>API: Structured data
        API->>DS: Query employee data
        DS->>API: Employee not found
        API->>LLM: Generate error response
        LLM->>API: "Cannot find employee" message
        API->>UI: Error response
        UI->>U: Display error message
    end
    
    Note over U,DB: Error Path: Invalid Employee
```

### Functional Workflow Diagram
```mermaid
flowchart TD
    A[User Starts Chat] --> B[Extract Employee Info]
    B --> C{Valid Employee Data?}
    C -->|No| D[Request More Info]
    C -->|Yes| E[Verify Employee Identity]
    E --> F{Employee Found?}
    F -->|No| G[Display Error Message]
    F -->|Yes| H[Determine Intent]
    H --> I{Intent Type?}
    I -->|Query| J[Retrieve Benefits Data]
    I -->|Update| K[Check Eligibility]
    I -->|Unknown| L[Ask for Clarification]
    J --> M[Format Response]
    K --> N{Eligible for Change?}
    N -->|No| O[Explain Restrictions]
    N -->|Yes| P[Process Update Request]
    P --> Q[Confirm Changes]
    M --> R[Send Response]
    O --> R
    Q --> R
    G --> S[Ask for Retry]
    L --> S
    R --> T[Wait for Next Input]
    S --> T
    T --> U{Continue Chat?}
    U -->|Yes| B
    U -->|No| V[End Session]
    
    style A fill:#e1f5fe
    style V fill:#ffebee
    style G fill:#ffcdd2
    style O fill:#ffcdd2
```

### User Journey Flow Diagram
```mermaid
journey
    title Service Rep Handling Employee Benefits Call
    section Call Initiation
      Receive employee call: 3: Service Rep
      Open chat interface: 4: Service Rep
      
    section Employee Identification
      Ask for employee details: 3: Service Rep
      Enter "John Smith, ID 12345": 5: Service Rep
      System verifies identity: 5: System
      
    section Benefits Inquiry
      Employee asks about 401k: 4: Service Rep
      Enter "Check 401k contribution": 5: Service Rep
      System retrieves data: 5: System
      Display current rate 8%: 5: System
      
    section Update Request
      Employee wants to increase to 10%: 4: Service Rep
      Enter "Increase to 10%": 5: Service Rep
      System checks eligibility: 5: System
      Confirm change allowed: 5: System
      
    section Resolution
      Process update request: 5: System
      Confirm changes made: 5: System
      Provide summary to employee: 5: Service Rep
      End call successfully: 5: Service Rep
```

### System Component Interaction Diagram
```mermaid
graph LR
    subgraph "User Interface"
        UI[Streamlit Chat]
        UIC[Chat Components]
        UIS[Session State]
        UI --> UIC
        UI --> UIS
    end
    
    subgraph "Application Logic"
        AL[FastAPI App]
        ALC[Chat Controller]
        ALS[Session Manager]
        ALA[Agent Orchestrator]
        AL --> ALC
        AL --> ALS
        AL --> ALA
    end
    
    subgraph "AI Processing"
        AI[LLM Agent]
        AIP[Prompt Templates]
        AIR[Response Parser]
        AIN[NLP Processor]
        AI --> AIP
        AI --> AIR
        AI --> AIN
    end
    
    subgraph "Data Services"
        DS[Data Manager]
        DSE[Employee Service]
        DSB[Benefits Service]
        DSR[Rules Engine]
        DS --> DSE
        DS --> DSB
        DS --> DSR
    end
    
    subgraph "External Systems"
        ES[Mock APIs]
        ESE[Employee API]
        ESB[Benefits API]
        ESV[Validation API]
        ES --> ESE
        ES --> ESB
        ES --> ESV
    end
    
    UI -->|HTTP| AL
    AL -->|LLM Calls| AI
    AL -->|Data Requests| DS
    DS -->|API Calls| ES
    
    style UI fill:#e3f2fd
    style AL fill:#f3e5f5
    style AI fill:#e8f5e8
    style DS fill:#fff3e0
    style ES fill:#fce4ec
```

### Data Structure
```json
{
  "employee": {
    "id": "12345",
    "name": "John Smith",
    "dob": "1990-05-15",
    "ssn": "xxx-xx-1234",
    "employment_status": "full_time",
    "hire_date": "2020-03-01"
  },
  "benefits": {
    "401k": {
      "contribution_rate": 8,
      "employer_match": 4,
      "balance": 45000,
      "eligible_for_match": true
    },
    "medical": {
      "plan": "PPO Gold",
      "coverage": "employee_only",
      "eligible": true
    }
  }
}
```

### Eligibility Rules
- **Medical**: Full-time employees immediately eligible; Part-time after 1000 hours or 1 year
- **401k Match**: All employees must wait 1 year before employer match contribution

## Setup Instructions

### Local LLM Setup (Ollama)
```bash
# Install Ollama (macOS)
curl -fsSL https://ollama.com/install.sh | sh

# Pull Llama 3.1 8B model
ollama pull llama3.1:8b

# Verify installation
ollama run llama3.1:8b
```

### Application Setup
```bash
# Clone repository
git clone [repo-url]
cd ai-service-center-poc

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start FastAPI backend
uvicorn main:app --reload --port 8000

# Start Streamlit frontend
streamlit run chat_app.py
```

## Risk Mitigation

### Technical Risks
- **LLM Accuracy**: Implement strict validation and "I don't know" responses
- **API Failures**: Graceful error handling with user-friendly messages
- **Data Security**: Mock data only, no real PI/HIPAA data in POC

### Demo Risks
- **Backup Plan**: Keep mocked APIs as fallback if real API integration fails
- **Performance**: Local LLM ensures no network dependencies during demo
- **Environment**: Self-contained MacBook Pro setup eliminates external dependencies

## Success Metrics
- **Accuracy**: 95%+ correct responses for demo scenarios
- **User Experience**: Natural conversation flow with <3 second response times
- **Error Handling**: Graceful failure modes with appropriate messaging
- **Stakeholder Confidence**: Clear demonstration of concept viability

## Future Roadmap
1. **Phase 3**: Integration with actual REST APIs
2. **Phase 4**: Voice interface capability
3. **Phase 5**: Additional benefit types (dental, vision, disability)
4. **Phase 6**: Multi-language support
5. **Phase 7**: Advanced analytics and reporting

## Deliverables
- Working Streamlit chat application
- FastAPI backend with mocked APIs
- Comprehensive setup documentation
- Demo script with test scenarios
- Technical documentation and architecture diagrams

---
**Timeline**: 3-5 days | **Team**: 1-2 developers | **Environment**: MacBook Pro (local)