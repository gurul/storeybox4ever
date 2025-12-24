# StoreyBox Diagrams (Mermaid.js)

## 1. Complete Session Flow

```mermaid
flowchart TD
    subgraph IDLE["🏠 IDLE STATE"]
        I1[Live Camera Feed]
        I2["'step inside'"]
        I3[Face Detection Active]
    end

    subgraph WAKE["⚡ WAKE"]
        W1[Face Detected]
        W2[User Taps Screen]
        W3[🔊 Signature Sound Plays]
    end

    subgraph STORY["📖 STORY MODE"]
        direction TB
        S1[Show 4 Prompts]
        S2{20s Timer}
        S3[User Taps Prompt]
        S4[Auto-Select Random]
        S5[Start 60s Capture]
        S6[Stream to AI]
        S7{50s Warning}
        S8[Soft Warning Display]
        S9{60s Reached}
        S10[Hard Stop]
        S11[Animation Overlay]
        S12[AI Picks Next 4 Prompts]
        S13{Moment 4 Complete?}
        S14[AI Frame Selection]
        S15[AI Vibe Summary]
        S16[AI Background Generation]
    end

    subgraph CONSENT["✅ CONSENT"]
        C1[One Person Consents for Group]
        C2[Private Only]
        C3[Link Shareable]
        C4[Network Visible]
        C5[Retention Choice]
    end

    subgraph OUTPUT["🖨️ OUTPUT"]
        O1[Generate Strip Image]
        O2[Add QR Code]
        O3["How Many Prints? 1-4"]
        O4[Send to Printer]
        O5{Print Success?}
        O6[Show URL on Screen]
        O7[Print Complete]
    end

    subgraph COMPLETE["👋 COMPLETE"]
        CO1[Show URL]
        CO2["'Thanks for sharing'"]
        CO3[10s Timeout]
    end

    subgraph CLEANUP["🧹 CLEANUP"]
        CL1[Delete Local Files]
        CL2[Return to Idle]
    end

    subgraph ABANDON["⚠️ ABANDONMENT"]
        A1{30s No Face + No Audio?}
        A2[Cancel Button Pressed?]
        A3[Wipe Session]
    end

    %% Main Flow
    IDLE --> W1
    W1 --> W2
    W2 --> W3
    W3 --> S1

    %% Story Mode Flow
    S1 --> S2
    S2 -->|Timeout| S4
    S2 -->|User Taps| S3
    S3 --> S5
    S4 --> S5
    S5 --> S6
    S6 --> S7
    S7 -->|Yes| S8
    S8 --> S9
    S7 -->|No| S9
    S9 -->|Yes| S10
    S10 --> S11
    S11 --> S12
    S12 --> S13
    S13 -->|No| S1
    S13 -->|Yes| S14
    S14 --> S15
    S15 --> S16
    S16 --> CONSENT

    %% Consent Flow
    C1 --> C2
    C1 --> C3
    C1 --> C4
    C1 --> C5
    CONSENT --> O1

    %% Output Flow
    O1 --> O2
    O2 --> O3
    O3 --> O4
    O4 --> O5
    O5 -->|No| O6
    O5 -->|Yes| O7
    O6 --> COMPLETE
    O7 --> COMPLETE

    %% Complete Flow
    CO1 --> CO2
    CO2 --> CO3
    CO3 --> CLEANUP
    CL1 --> CL2
    CL2 --> IDLE

    %% Abandonment (can happen from multiple states)
    S5 -.-> A1
    S1 -.-> A1
    A1 -->|Yes| A3
    A2 --> A3
    A3 --> IDLE

    %% Styling
    classDef idle fill:#e8f5e9,stroke:#4caf50
    classDef story fill:#e3f2fd,stroke:#2196f3
    classDef consent fill:#f3e5f5,stroke:#9c27b0
    classDef output fill:#fce4ec,stroke:#e91e63
    classDef abandon fill:#ffebee,stroke:#f44336

    class IDLE idle
    class STORY story
    class CONSENT consent
    class OUTPUT output
    class ABANDON abandon
```

---

## 2. State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> PromptSelection1: Face + Tap

    %% Story Mode States
    state "Story Mode" as StoryMode {
        PromptSelection1: Prompt Selection (1)
        StoryCapture1: Capture (1)
        Analyzing1: Analyzing (1)
        
        PromptSelection2: Prompt Selection (2)
        StoryCapture2: Capture (2)
        Analyzing2: Analyzing (2)
        
        PromptSelection3: Prompt Selection (3)
        StoryCapture3: Capture (3)
        Analyzing3: Analyzing (3)
        
        PromptSelection4: Prompt Selection (4)
        StoryCapture4: Capture (4)
        
        PromptSelection1 --> StoryCapture1: Select/Timeout
        StoryCapture1 --> Analyzing1: 60s
        Analyzing1 --> PromptSelection2: AI Ready
        
        PromptSelection2 --> StoryCapture2: Select/Timeout
        StoryCapture2 --> Analyzing2: 60s
        Analyzing2 --> PromptSelection3: AI Ready
        
        PromptSelection3 --> StoryCapture3: Select/Timeout
        StoryCapture3 --> Analyzing3: 60s
        Analyzing3 --> PromptSelection4: AI Ready
        
        PromptSelection4 --> StoryCapture4: Select/Timeout
        StoryCapture4 --> FrameSelection: 60s
    }

    %% End States
    FrameSelection: Frame Selection (AI)
    BackgroundGen: Background Generation (AI)
    Consent: Consent Screen
    StripGeneration: Strip Generation
    PrintCount: Print Count Selection
    Printing: Printing
    Complete: Complete Screen
    Cleanup: Cleanup

    FrameSelection --> BackgroundGen
    BackgroundGen --> Consent
    Consent --> StripGeneration
    
    StripGeneration --> PrintCount
    PrintCount --> Printing
    Printing --> Complete
    Complete --> Cleanup: 10s
    Cleanup --> Idle

    %% Error & Abandonment
    state "Error/Abandon" as ErrorStates {
        Abandoned: Abandoned
        NetworkError: Network Error
        PrinterError: Printer Error
    }
    
    StoryCapture1 --> Abandoned: 30s Inactive
    StoryCapture2 --> Abandoned: 30s Inactive
    StoryCapture3 --> Abandoned: 30s Inactive
    StoryCapture4 --> Abandoned: 30s Inactive
    
    Abandoned --> Idle: Wipe
    NetworkError --> Idle: Wipe
    PrinterError --> Complete: Show URL
```

---

## 3. Software Architecture

```mermaid
flowchart TB
    subgraph APP["📱 APPLICATION LAYER"]
        direction LR
        IdleScreen[Idle Screen]
        PromptSelect[Prompt Select]
        Capture[Capture Screen]
        Analyzing[Analyzing Screen]
        ConsentUI[Consent Screen]
        PrintCount[Print Count]
        PrintingUI[Printing Screen]
        CompleteUI[Complete Screen]
        ErrorUI[Error Screen]
    end

    subgraph ORCH["🎛️ ORCHESTRATION LAYER"]
        SessionController[Session Controller]
        StateManager[State Manager]
        TimerManager[Timer Manager]
    end

    subgraph ENGINES["⚙️ ENGINE LAYER"]
        direction TB
        subgraph Capture_Engines["Capture"]
            CaptureEngine[Capture Engine]
            PresenceEngine[Presence Engine]
        end
        subgraph Content_Engines["Content"]
            PromptEngine[Prompt Engine]
            AudioEngine[Audio Engine]
        end
        subgraph Output_Engines["Output"]
            FrameEngine[Frame Engine]
            BackgroundEngine[Background Engine]
            StripEngine[Strip Engine]
            PrintEngine[Print Engine]
        end
        subgraph Network_Engines["Network"]
            UploadEngine[Upload Engine]
            APIClient[API Client]
        end
    end

    subgraph AI["🤖 AI LAYER"]
        AIService[AI Service]
        ClaudeAPI[Claude API]
        GeminiAPI[Gemini Vision]
        ImageGenAPI[Image Gen API]
    end

    subgraph STORAGE["💾 STORAGE LAYER"]
        direction LR
        LocalTemp[Local Temp Files]
        ConfigStore[Config Store]
        PromptLibrary[Prompt Library]
    end

    subgraph CLOUD["☁️ CLOUD"]
        direction LR
        S3[S3/R2 Bucket]
        Vercel[Vercel API]
        StoryPage[Story Page]
    end

    subgraph HARDWARE["🔌 HARDWARE"]
        direction LR
        Camera[Blackmagic 6K Pro]
        Mic[Sennheiser MKE 600]
        Tascam[Tascam DR-701D]
        Printer[DNP DS-RX1HS]
        Display[Touch Display]
    end

    %% Connections
    APP --> ORCH
    ORCH --> ENGINES
    ENGINES --> AI
    ENGINES --> STORAGE
    ENGINES --> CLOUD
    ENGINES --> HARDWARE

    CaptureEngine --> Camera
    CaptureEngine --> Mic
    Mic --> Tascam
    PrintEngine --> Printer
    APP --> Display

    AIService --> ClaudeAPI
    AIService --> GeminiAPI
    AIService --> ImageGenAPI

    UploadEngine --> S3
    APIClient --> Vercel
    Vercel --> StoryPage

    PromptEngine --> PromptLibrary
    CaptureEngine --> LocalTemp
    StripEngine --> LocalTemp
```

---

## 4. Data Flow

```mermaid
flowchart LR
    subgraph INPUT["📥 INPUT"]
        Camera[🎥 Camera]
        Mic[🎤 Microphone]
        Touch[👆 Touch Input]
    end

    subgraph CAPTURE["🎬 CAPTURE"]
        VideoStream[Video Stream]
        AudioStream[Audio Stream]
        LocalMOV[Local .mov Files]
    end

    subgraph AI_PROCESS["🤖 AI PROCESSING"]
        PromptAI[Prompt Selection AI]
        FrameAI[Frame Selection AI]
        VibeAI[Vibe Summary AI]
        BackgroundAI[Background Gen AI]
    end

    subgraph GENERATION["🎨 GENERATION"]
        Frames[Selected Frames]
        Background[Generated Background]
        QRCode[QR Code]
        StripImage[Strip Image]
    end

    subgraph OUTPUT["📤 OUTPUT"]
        Printer[🖨️ Printer]
        CloudStorage[☁️ Cloud Storage]
        WebPage[🌐 Story Page]
    end

    Camera --> VideoStream
    Mic --> AudioStream
    VideoStream --> LocalMOV
    AudioStream --> LocalMOV
    
    LocalMOV --> PromptAI
    PromptAI --> |"Next 4 Prompts"| Touch
    
    LocalMOV --> FrameAI
    FrameAI --> Frames
    
    LocalMOV --> VibeAI
    VibeAI --> BackgroundAI
    BackgroundAI --> Background
    
    Frames --> StripImage
    Background --> StripImage
    QRCode --> StripImage
    
    StripImage --> Printer
    LocalMOV --> CloudStorage
    Frames --> CloudStorage
    StripImage --> CloudStorage
    CloudStorage --> WebPage
```

---

## 5. Prompt Selection Flow

```mermaid
flowchart TD
    subgraph LIBRARY["📚 PROMPT LIBRARY"]
        P1[50-150 Curated Prompts]
        Tags[Soft Tags: depth, energy, direction, context]
    end

    subgraph CONTEXT["📋 SESSION CONTEXT"]
        MomentNum[Current Moment: 1-4]
        LastPrompt[Previous Prompt]
        LastResponse[Previous Response Video/Audio]
        GroupType[Detected: solo/couple/friends/family]
    end

    subgraph RULES["📏 HARD RULES"]
        R1[Moment 1: depth 1-2 only]
        R2[Moment 4: grounding only]
        R3[Never jump depth > +1]
        R4[Default to lighter if uncertain]
    end

    subgraph AI_SELECT["🤖 AI SELECTION"]
        Filter[Filter Candidates by Rules]
        Analyze[Analyze Last Response]
        Match[Match Energy + Depth]
        Select[Select Best 4]
    end

    subgraph OUTPUT["📤 OUTPUT"]
        FourPrompts[4 Prompt Options]
        Display[Display to User]
        Timer[20s Timer]
        AutoSelect[Auto-Select if Timeout]
    end

    LIBRARY --> Filter
    CONTEXT --> Analyze
    RULES --> Filter
    Filter --> Match
    Analyze --> Match
    Match --> Select
    Select --> FourPrompts
    FourPrompts --> Display
    Display --> Timer
    Timer --> AutoSelect
```

---

## 6. Strip Generation

```mermaid
flowchart TD
    subgraph INPUTS["📥 INPUTS"]
        Frame1[Frame 1]
        Frame2[Frame 2]
        Frame3[Frame 3]
        Frame4[Frame 4]
        BG[AI Background]
    end

    subgraph LAYOUT["📐 LAYOUT"]
        direction TB
        Header["'storeybox'"]
        F1Box[Frame 1 Area]
        F2Box[Frame 2 Area]
        F3Box[Frame 3 Area]
        F4Box[Frame 4 Area]
        Footer["'take a moment'"]
        QR[QR Code]
    end

    subgraph COMPOSITE["🎨 COMPOSITE"]
        ApplyBG[Apply Background]
        PlaceFrames[Place 4 Frames]
        AddBranding[Add Branding Text]
        RenderFinal[Render Final Image]
    end

    subgraph OUTPUT["📤 OUTPUT"]
        StripPNG[strip.png]
        ToPrinter[Send to Printer]
        ToCloud[Upload to Cloud]
    end

    Frame1 --> F1Box
    Frame2 --> F2Box
    Frame3 --> F3Box
    Frame4 --> F4Box
    BG --> ApplyBG

    ApplyBG --> PlaceFrames
    F1Box --> PlaceFrames
    F2Box --> PlaceFrames
    F3Box --> PlaceFrames
    F4Box --> PlaceFrames
    
    PlaceFrames --> AddBranding
    Header --> AddBranding
    Footer --> AddBranding
    
    AddBranding --> QR
    QR --> RenderFinal
    
    RenderFinal --> StripPNG
    StripPNG --> ToPrinter
    StripPNG --> ToCloud
```

---

## 7. Error Handling

```mermaid
flowchart TD
    subgraph ERRORS["⚠️ ERROR TYPES"]
        E1[Network Lost]
        E2[AI Service Failed]
        E3[Printer Failed]
        E4[Upload Failed]
        E5[Abandonment Detected]
    end

    subgraph HANDLERS["🔧 HANDLERS"]
        H1[Show Error Screen]
        H2[Retry 2x]
        H3[Use Fallback]
        H4[Retry 3x with Backoff]
        H5[Silent Wipe]
    end

    subgraph FALLBACKS["🔄 FALLBACKS"]
        F1[Random Prompt Selection]
        F2[Solid Color Background]
        F3[Show URL Instead]
    end

    subgraph RECOVERY["✅ RECOVERY"]
        R1[Return to Idle]
        R2[Continue Session]
        R3[Complete with Degradation]
    end

    E1 --> H1 --> R1
    
    E2 --> H2
    H2 -->|Success| R2
    H2 -->|Fail| H3
    H3 --> F1
    H3 --> F2
    F1 --> R2
    F2 --> R2

    E3 --> F3 --> R3

    E4 --> H4
    H4 -->|Success| R3
    H4 -->|Fail| H1 --> R1

    E5 --> H5 --> R1
```

---

## 8. Consent Flow

```mermaid
flowchart TD
    subgraph ENTRY["📍 ENTRY"]
        MomentsComplete[4 Moments Complete]
        FramesSelected[Frames Selected]
        BackgroundGenerated[Background Generated]
    end

    subgraph CONSENT_UI["✅ CONSENT SCREEN"]
        Preview[Preview Strip]
        ConsentPrompt["One person consent for group"]
    end

    subgraph OPTIONS["📋 OPTIONS"]
        direction TB
        ShareLevel[Share Level]
        SL1[🔒 Private - QR Only]
        SL2[🔗 Link Shareable]
        SL3[🌐 Network Visible]
        
        Retention[Retention]
        RT1[♾️ Forever]
        RT2[📅 One Year]
        RT3[🗓️ One Month]
        RT4[👁️ Delete After Viewing]
    end

    subgraph TIMEOUT["⏱️ TIMEOUT HANDLING"]
        Timer[Waiting for Input]
        Abandoned{User Left?}
        DefaultPrivate[Default to Most Private]
    end

    subgraph OUTPUT["📤 OUTPUT"]
        ConsentRecord[Consent Record Created]
        ProceedToStrip[Proceed to Strip Gen]
    end

    ENTRY --> Preview
    Preview --> ConsentPrompt
    ConsentPrompt --> ShareLevel
    ShareLevel --> SL1
    ShareLevel --> SL2
    ShareLevel --> SL3
    
    ConsentPrompt --> Retention
    Retention --> RT1
    Retention --> RT2
    Retention --> RT3
    Retention --> RT4

    SL1 --> ConsentRecord
    SL2 --> ConsentRecord
    SL3 --> ConsentRecord
    RT1 --> ConsentRecord
    RT2 --> ConsentRecord
    RT3 --> ConsentRecord
    RT4 --> ConsentRecord

    ConsentPrompt --> Timer
    Timer --> Abandoned
    Abandoned -->|Yes| DefaultPrivate
    DefaultPrivate --> ConsentRecord
    Abandoned -->|No| Timer

    ConsentRecord --> ProceedToStrip
```

---

## 9. Timeline View

```mermaid
gantt
    title StoreyBox Session Timeline (Story Mode)
    dateFormat ss
    axisFormat %S

    section Idle
    Waiting for user     :idle, 00, 5s

    section Wake
    Face detected        :wake, after idle, 1s
    Signature sound      :sound, after wake, 2s

    section Mode Select
    Choose mode          :mode, after sound, 3s

    section Moment 1
    Show prompts         :p1, after mode, 2s
    User selects (20s max) :s1, after p1, 5s
    Capture (60s)        :c1, after s1, 60s
    AI analyzing         :a1, after c1, 3s

    section Moment 2
    Show prompts         :p2, after a1, 2s
    User selects         :s2, after p2, 5s
    Capture (60s)        :c2, after s2, 60s
    AI analyzing         :a2, after c2, 3s

    section Moment 3
    Show prompts         :p3, after a2, 2s
    User selects         :s3, after p3, 5s
    Capture (60s)        :c3, after s3, 60s
    AI analyzing         :a3, after c3, 3s

    section Moment 4
    Show prompts         :p4, after a3, 2s
    User selects         :s4, after p4, 5s
    Capture (60s)        :c4, after s4, 60s

    section Processing
    Frame selection      :fs, after c4, 5s
    Background gen       :bg, after fs, 8s
    
    section Consent
    Consent screen       :con, after bg, 10s

    section Output
    Strip generation     :strip, after con, 3s
    Print count select   :count, after strip, 5s
    Printing             :print, after count, 10s

    section Complete
    Show URL             :complete, after print, 10s
    Cleanup              :clean, after complete, 2s
```

---

## 10. Cloud Architecture

```mermaid
flowchart TB
    subgraph BOX["📦 STOREY BOX"]
        App[StoreyBox.app]
        LocalFiles[Local Temp Files]
    end

    subgraph UPLOAD["📤 UPLOAD FLOW"]
        Stream[Stream During Capture]
        Presigned[Get Presigned URL]
        Upload[Upload to S3]
    end

    subgraph CLOUD_STORAGE["☁️ CLOUD STORAGE - S3/R2"]
        Bucket[storeybox-media]
        Sessions[/sessions/]
        SessionDir[/session-id/]
        Videos["moment-1.mp4, moment-2.mp4, moment-3.mp4, moment-4.mp4"]
        Frames["frame-1.jpg, frame-2.jpg, frame-3.jpg, frame-4.jpg"]
        Strip[strip.png]
        Meta[metadata.json]
    end

    subgraph VERCEL["⚡ VERCEL"]
        API[/api/]
        CreateSession["POST /api/sessions"]
        UpdateSession["PUT /api/sessions/id"]
        GetUploadURL["POST /api/sessions/id/upload"]
        MarkComplete["POST /api/sessions/id/complete"]
        
        Pages[/s/]
        StoryPage["GET /s/id"]
    end

    subgraph DATABASE["🗄️ DATABASE"]
        SessionsTable[Sessions Table]
        ConsentTable[Consent Table]
    end

    App --> Stream
    Stream --> Presigned
    Presigned --> GetUploadURL
    GetUploadURL --> Upload
    Upload --> Bucket
    
    Bucket --> Sessions --> SessionDir
    SessionDir --> Videos
    SessionDir --> Frames
    SessionDir --> Strip
    SessionDir --> Meta

    App --> CreateSession
    CreateSession --> SessionsTable
    App --> UpdateSession
    UpdateSession --> ConsentTable
    App --> MarkComplete

    StoryPage --> SessionsTable
    StoryPage --> Bucket

    subgraph USER["👤 USER"]
        QRScan[Scan QR Code]
        Browser[Browser]
    end

    QRScan --> Browser
    Browser --> StoryPage
```
