# Technical Stack and Schema Design

## Tech Stack

### Frontend
1. **Mobile**
   - React Native
   - Native Modules: 
     - react-native-geolocation
     - react-native-push-notification
     - react-native-maps
     - react-native-encrypted-storage
   - State Management: Redux Toolkit
   - Real-time: Socket.io-client
   - UI Components: 
     - react-native-elements
     - react-native-paper
     - react-native-vector-icons

2. **Web**
   - Next.js
   - TypeScript
   - TailwindCSS
   - WebSocket
   - PWA capabilities
   - Service Workers

### Backend
1. **API Layer**
   - Node.js
   - Express.js/NestJS
   - TypeScript
   - GraphQL (Apollo Server)
   - REST endpoints

2. **Authentication**
   - Passport.js
   - JWT
   - OAuth2
   - bcrypt

3. **Real-time**
   - Socket.io
   - Redis Pub/Sub
   - WebSocket

### Database
1. **Primary Database**
   - PostgreSQL 15+
   - TypeORM/Prisma
   - PostGIS for location

2. **Caching**
   - Redis
   - Redis Cluster

3. **Search**
   - Elasticsearch
   - elasticsearch-js client

### ML/AI
1. **Core ML**
   - Python
   - TensorFlow/PyTorch
   - scikit-learn
   - pandas

2. **NLP**
   - spaCy
   - NLTK
   - Transformers

### DevOps
1. **Infrastructure**
   - Docker
   - Kubernetes
   - Terraform
   - AWS/GCP

2. **Monitoring**
   - Prometheus
   - Grafana
   - ELK Stack

3. **CI/CD**
   - GitHub Actions
   - Jenkins
   - ArgoCD

## Database Schema Design

### User Schema
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    bio TEXT,
    avatar_url VARCHAR(255),
    location_sharing BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_active TIMESTAMP WITH TIME ZONE,
    preferences JSONB,
    settings JSONB
);

CREATE TABLE user_values (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    category VARCHAR(50) NOT NULL,
    value_name VARCHAR(100) NOT NULL,
    strength INTEGER CHECK (strength BETWEEN 0 AND 100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, category, value_name)
);
```

### Location Schema
```sql
CREATE TABLE locations (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    latitude DECIMAL(10,8) NOT NULL,
    longitude DECIMAL(11,8) NOT NULL,
    accuracy DECIMAL(5,2),
    location_type VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE
);

CREATE TABLE places (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    latitude DECIMAL(10,8) NOT NULL,
    longitude DECIMAL(11,8) NOT NULL,
    place_type VARCHAR(50),
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### Connection Schema
```sql

-- The heart of the system
CREATE TABLE connection_intelligence (
    id UUID PRIMARY KEY,
    question_text TEXT NOT NULL,
    context_triggers JSONB, -- when to ask
    depth_level INTEGER,    -- surface to deep
    success_patterns JSONB, -- what worked
    emotional_markers JSONB -- emotional context
);

-- Learning from interactions
CREATE TABLE interaction_learning (
    id UUID PRIMARY KEY,
    interaction_id UUID,
    what_worked BOOLEAN,
    emotional_response JSONB,
    connection_depth INTEGER,
    growth_indicators JSONB
);

-- Growth pathways
CREATE TABLE growth_paths (
    id UUID PRIMARY KEY,
    user_id UUID,
    current_stage JSONB,
    next_steps JSONB,
    recommendations JSONB
);


CREATE TABLE connections (
    id UUID PRIMARY KEY,
    user1_id UUID REFERENCES users(id),
    user2_id UUID REFERENCES users(id),
    status VARCHAR(50) NOT NULL,
    strength DECIMAL(5,2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_interaction TIMESTAMP WITH TIME ZONE,
    metadata JSONB,
    CHECK (user1_id < user2_id)
);

CREATE TABLE connection_interactions (
    id UUID PRIMARY KEY,
    connection_id UUID REFERENCES connections(id),
    interaction_type VARCHAR(50) NOT NULL,
    quality_score DECIMAL(3,2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);
```

### Group Schema
```sql
CREATE TABLE groups (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(50),
    settings JSONB
);

CREATE TABLE group_members (
    id UUID PRIMARY KEY,
    group_id UUID REFERENCES groups(id),
    user_id UUID REFERENCES users(id),
    role VARCHAR(50),
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(group_id, user_id)
);
```

### Chat Schema
```sql
CREATE TABLE chats (
    id UUID PRIMARY KEY,
    chat_type VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);

CREATE TABLE messages (
    id UUID PRIMARY KEY,
    chat_id UUID REFERENCES chats(id),
    sender_id UUID REFERENCES users(id),
    content TEXT,
    message_type VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);
```

### Activity Schema
```sql
CREATE TABLE activities (
    id UUID PRIMARY KEY,
    activity_type VARCHAR(50) NOT NULL,
    user_id UUID REFERENCES users(id),
    related_id UUID,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);

CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    type VARCHAR(50) NOT NULL,
    content JSONB NOT NULL,
    read BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE
);
```

### Analytics Schema
```sql
CREATE TABLE metrics (
    id UUID PRIMARY KEY,
    metric_type VARCHAR(50) NOT NULL,
    value DECIMAL(15,5) NOT NULL,
    dimensions JSONB,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE events (
    id UUID PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    user_id UUID REFERENCES users(id),
    properties JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

## Key Indices
```sql
-- User indices
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_last_active ON users(last_active);

-- Location indices
CREATE INDEX idx_locations_user_latest ON locations(user_id, created_at DESC);
CREATE INDEX idx_locations_spatial ON locations USING GIST (
    ST_SetSRID(ST_MakePoint(longitude, latitude), 4326)
);

-- Connection indices
CREATE INDEX idx_connections_users ON connections(user1_id, user2_id);
CREATE INDEX idx_connections_status ON connections(status);

-- Message indices
CREATE INDEX idx_messages_chat_time ON messages(chat_id, created_at DESC);

-- Activity indices
CREATE INDEX idx_activities_user_time ON activities(user_id, created_at DESC);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id) WHERE NOT read;
```

## Important Notes

1. **Performance Considerations**
   - Use appropriate index strategies
   - Implement table partitioning for large tables
   - Regular vacuum and analyze
   - Cache frequently accessed data

2. **Security Measures**
   - Encrypt sensitive data
   - Implement Row Level Security
   - Regular security audits
   - Backup strategies

3. **Scalability**
   - Horizontal scaling preparation
   - Sharding strategies
   - Connection pooling
   - Query optimization

Would you like me to elaborate on any specific aspect of the tech stack or schema design?