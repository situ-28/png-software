# Production-Ready DVR → HLS → Web System Implementation Guide

## PROJECT GOAL
Build a robust, scalable, secure production system that manages multiple Hikvision DVRs (ONVIF/RTSP), converts RTSP streams to HLS using ffmpeg with automatic failover, serves adaptive bitrate streams, provides comprehensive monitoring/alerting, and includes a React dashboard with role-based access control.

---

## 1) PRODUCTION TECHNOLOGY STACK

### Backend Architecture
* **Runtime:** Node.js 18+ LTS with TypeScript for type safety
* **Framework:** Express.js with async/await error handling
* **Database:** PostgreSQL 14+ with connection pooling (`pg-pool`)
* **Cache:** Redis for session management, stream metadata, and rate limiting
* **Process Management:** PM2 Cluster mode with auto-restart
* **Message Queue:** Bull Queue (Redis-based) for background job processing
* **Authentication:** JWT with refresh tokens + OAuth2 integration
* **API Documentation:** OpenAPI 3.0 (Swagger)

### Libraries & Dependencies
```json
{
  "express": "^4.18.0",
  "typescript": "^5.0.0",
  "@types/node": "^18.0.0",
  "pg": "^8.8.0",
  "pg-pool": "^3.6.0",
  "redis": "^4.5.0",
  "bull": "^4.10.0",
  "bcrypt": "^5.1.0",
  "jsonwebtoken": "^9.0.0",
  "helmet": "^6.0.0",
  "rate-limiter-flexible": "^2.4.0",
  "joi": "^17.7.0",
  "winston": "^3.8.0",
  "prometheus-client": "^14.1.0",
  "onvif": "^0.6.0",
  "node-cron": "^3.0.0"
}
```

### Infrastructure & DevOps
* **Container:** Docker multi-stage builds + Docker Compose
* **Reverse Proxy:** NGINX with SSL termination, rate limiting, caching
* **Load Balancer:** HAProxy or AWS ALB with health checks
* **Monitoring:** Prometheus + Grafana + AlertManager
* **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)
* **Secrets:** HashiCorp Vault or AWS Secrets Manager
* **CI/CD:** GitHub Actions or GitLab CI with automated testing
* **Environment:** Kubernetes or AWS ECS for orchestration

---

## 2) ENHANCED REPOSITORY STRUCTURE

```
dvr-hls-production/
├─ backend/
│  ├─ src/
│  │  ├─ controllers/
│  │  │  ├─ dvrController.ts
│  │  │  ├─ streamController.ts
│  │  │  ├─ authController.ts
│  │  │  └─ healthController.ts
│  │  ├─ services/
│  │  │  ├─ ffmpegManager.ts
│  │  │  ├─ onvifService.ts
│  │  │  ├─ streamingService.ts
│  │  │  ├─ authService.ts
│  │  │  ├─ notificationService.ts
│  │  │  └─ metricsService.ts
│  │  ├─ models/
│  │  │  ├─ User.ts
│  │  │  ├─ DVR.ts
│  │  │  ├─ Stream.ts
│  │  │  └─ AuditLog.ts
│  │  ├─ middleware/
│  │  │  ├─ auth.ts
│  │  │  ├─ validation.ts
│  │  │  ├─ rateLimiting.ts
│  │  │  ├─ errorHandler.ts
│  │  │  └─ requestLogger.ts
│  │  ├─ utils/
│  │  │  ├─ crypto.ts
│  │  │  ├─ logger.ts
│  │  │  ├─ validators.ts
│  │  │  └─ constants.ts
│  │  ├─ routes/
│  │  │  ├─ v1/
│  │  │  │  ├─ dvrs.ts
│  │  │  │  ├─ streams.ts
│  │  │  │  ├─ auth.ts
│  │  │  │  └─ admin.ts
│  │  ├─ jobs/
│  │  │  ├─ streamHealthCheck.ts
│  │  │  ├─ cleanupOldSegments.ts
│  │  │  └─ generateReports.ts
│  │  ├─ config/
│  │  │  ├─ database.ts
│  │  │  ├─ redis.ts
│  │  │  └─ app.ts
│  │  └─ server.ts
│  ├─ db/
│  │  ├─ migrations/
│  │  │  ├─ 001_initial_schema.sql
│  │  │  ├─ 002_add_users_rbac.sql
│  │  │  └─ 003_add_audit_logging.sql
│  │  └─ seeds/
│  ├─ tests/
│  │  ├─ unit/
│  │  ├─ integration/
│  │  └─ e2e/
│  ├─ streams/                    // HLS output (NFS/S3 in production)
│  ├─ logs/
│  ├─ Dockerfile
│  ├─ docker-compose.yml
│  ├─ package.json
│  ├─ tsconfig.json
│  └─ README.md
├─ frontend/
│  ├─ src/
│  │  ├─ components/
│  │  │  ├─ common/
│  │  │  ├─ auth/
│  │  │  ├─ dvr/
│  │  │  │  ├─ DVRList.tsx
│  │  │  │  ├─ DVRForm.tsx
│  │  │  │  └─ DVRMetrics.tsx
│  │  │  └─ streaming/
│  │  │     ├─ AdaptiveHLSPlayer.tsx
│  │  │     ├─ MultiStreamView.tsx
│  │  │     └─ StreamControls.tsx
│  │  ├─ pages/
│  │  │  ├─ Dashboard.tsx
│  │  │  ├─ DVRManagement.tsx
│  │  │  ├─ LiveStreams.tsx
│  │  │  ├─ Analytics.tsx
│  │  │  └─ Settings.tsx
│  │  ├─ hooks/
│  │  ├─ services/
│  │  ├─ store/ (Redux Toolkit)
│  │  └─ utils/
├─ infrastructure/
│  ├─ docker/
│  ├─ kubernetes/
│  │  ├─ manifests/
│  │  └─ helm/
│  ├─ nginx/
│  │  ├─ nginx.conf
│  │  └─ ssl/
│  ├─ monitoring/
│  │  ├─ prometheus/
│  │  ├─ grafana/
│  │  └─ alertmanager/
│  ├─ terraform/ (or CloudFormation)
│  └─ scripts/
├─ docs/
│  ├─ api/
│  ├─ deployment/
│  └─ troubleshooting/
└─ .github/workflows/
```

---

## 3) PRODUCTION DATABASE SCHEMA

```sql
-- Enhanced schema with RBAC, audit logging, and performance optimizations
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Users and RBAC
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    permissions JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role_id INTEGER REFERENCES roles(id),
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Enhanced DVR table
CREATE TABLE dvrs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name TEXT NOT NULL,
    description TEXT,
    brand VARCHAR(50),
    model VARCHAR(100),
    firmware_version VARCHAR(50),
    ip INET NOT NULL,
    port INTEGER DEFAULT 554,
    username VARCHAR(255) NOT NULL,
    password_encrypted TEXT NOT NULL,
    rtsp_paths JSONB, -- Support multiple streams/channels
    onvif_port INTEGER DEFAULT 80,
    onvif_wsdl_url TEXT,
    capabilities JSONB,
    location JSONB, -- Geographic info
    timezone VARCHAR(50) DEFAULT 'UTC',
    status VARCHAR(20) DEFAULT 'offline' CHECK (status IN ('online', 'offline', 'error', 'maintenance')),
    last_heartbeat TIMESTAMP,
    health_score INTEGER DEFAULT 100 CHECK (health_score BETWEEN 0 AND 100),
    metadata JSONB,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Stream profiles for adaptive bitrate
CREATE TABLE stream_profiles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100) NOT NULL,
    resolution VARCHAR(20), -- 1080p, 720p, 480p
    bitrate INTEGER, -- kbps
    framerate INTEGER,
    codec VARCHAR(20),
    ffmpeg_params JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Enhanced streams table
CREATE TABLE streams (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    dvr_id UUID NOT NULL REFERENCES dvrs(id) ON DELETE CASCADE,
    profile_id UUID REFERENCES stream_profiles(id),
    channel_number INTEGER DEFAULT 1,
    stream_type VARCHAR(20) DEFAULT 'main' CHECK (stream_type IN ('main', 'sub', 'mobile')),
    ffmpeg_pid INTEGER,
    container_id VARCHAR(100), -- Docker container ID if applicable
    rtsp_url TEXT,
    hls_playlist_url TEXT,
    dash_playlist_url TEXT,
    output_directory TEXT,
    status VARCHAR(20) DEFAULT 'stopped' CHECK (status IN ('starting', 'running', 'stopping', 'stopped', 'error')),
    error_count INTEGER DEFAULT 0,
    last_error TEXT,
    bandwidth_usage BIGINT DEFAULT 0, -- bytes
    viewer_count INTEGER DEFAULT 0,
    uptime_seconds INTEGER DEFAULT 0,
    quality_metrics JSONB,
    started_by UUID REFERENCES users(id),
    started_at TIMESTAMP,
    stopped_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Stream analytics
CREATE TABLE stream_analytics (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    stream_id UUID REFERENCES streams(id) ON DELETE CASCADE,
    timestamp TIMESTAMP DEFAULT NOW(),
    viewer_count INTEGER,
    bandwidth_mbps DECIMAL,
    quality_score INTEGER,
    buffer_health INTEGER,
    error_rate DECIMAL,
    metadata JSONB
);

-- Audit logging
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    details JSONB,
    ip_address INET,
    user_agent TEXT,
    timestamp TIMESTAMP DEFAULT NOW()
);

-- Notifications/Alerts
CREATE TABLE alert_rules (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(200) NOT NULL,
    condition_type VARCHAR(50), -- 'stream_down', 'high_error_rate', 'disk_usage'
    threshold_config JSONB,
    notification_channels JSONB,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    rule_id UUID REFERENCES alert_rules(id),
    severity VARCHAR(20) CHECK (severity IN ('low', 'medium', 'high', 'critical')),
    title VARCHAR(500),
    description TEXT,
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'acknowledged', 'resolved')),
    acknowledged_by UUID REFERENCES users(id),
    acknowledged_at TIMESTAMP,
    resolved_at TIMESTAMP,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_dvrs_status ON dvrs(status);
CREATE INDEX idx_dvrs_ip ON dvrs(ip);
CREATE INDEX idx_streams_dvr_id ON streams(dvr_id);
CREATE INDEX idx_streams_status ON streams(status);
CREATE INDEX idx_streams_started_at ON streams(started_at);
CREATE INDEX idx_analytics_stream_timestamp ON stream_analytics(stream_id, timestamp);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
CREATE INDEX idx_audit_logs_user_action ON audit_logs(user_id, action);

-- Default roles
INSERT INTO roles (name, permissions) VALUES 
('admin', '{"dvr": ["create", "read", "update", "delete"], "stream": ["create", "read", "update", "delete"], "user": ["create", "read", "update", "delete"], "system": ["read", "update"]}'),
('operator', '{"dvr": ["read", "update"], "stream": ["create", "read", "update", "delete"], "user": ["read"]}'),
('viewer', '{"dvr": ["read"], "stream": ["read"]}');

-- Default stream profiles
INSERT INTO stream_profiles (name, resolution, bitrate, framerate, codec, ffmpeg_params) VALUES
('High Quality', '1080p', 4000, 30, 'h264', '{"preset": "medium", "crf": 20}'),
('Medium Quality', '720p', 2000, 30, 'h264', '{"preset": "fast", "crf": 23}'),
('Low Quality', '480p', 800, 25, 'h264', '{"preset": "veryfast", "crf": 26}'),
('Mobile', '360p', 400, 20, 'h264', '{"preset": "ultrafast", "crf": 28}');
```

---

## 4) PRODUCTION ENVIRONMENT VARIABLES

```bash
# Application
NODE_ENV=production
PORT=8000
API_VERSION=v1
CLUSTER_MODE=true
WORKER_PROCESSES=0  # 0 = auto-detect CPU cores

# Database
DATABASE_URL=postgresql://username:password@localhost:5432/dvr_production
DATABASE_POOL_MIN=2
DATABASE_POOL_MAX=20
DATABASE_SSL=true

# Redis
REDIS_URL=redis://localhost:6379
REDIS_CLUSTER_NODES=redis1:6379,redis2:6379,redis3:6379
REDIS_PASSWORD=secure_redis_password

# Security
ENCRYPTION_KEY=base64_encoded_32_byte_key
JWT_SECRET=secure_jwt_secret_256_bits
JWT_REFRESH_SECRET=secure_refresh_secret
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
SESSION_SECRET=secure_session_secret
BCRYPT_ROUNDS=12

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000  # 15 minutes
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_REDIS_PREFIX=rl_

# Streaming
HLS_ROOT=/data/streams
HLS_SEGMENT_DURATION=2
HLS_PLAYLIST_SIZE=5
HLS_CLEANUP_AGE=3600  # seconds
FFMPEG_PATH=/usr/bin/ffmpeg
FFMPEG_THREADS=4
FFMPEG_LOG_LEVEL=warning
ADAPTIVE_BITRATE=true
AUTO_RESTART_STREAMS=true

# Storage
STORAGE_TYPE=local  # local, s3, nfs
AWS_S3_BUCKET=dvr-hls-streams
AWS_S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Monitoring & Logging
LOG_LEVEL=info
LOG_FORMAT=json
METRICS_ENABLED=true
PROMETHEUS_PORT=9090
HEALTH_CHECK_INTERVAL=30000
STREAM_HEALTH_CHECK_INTERVAL=10000

# Notifications
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=alerts@example.com
SMTP_PASSWORD=smtp_password
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
WEBHOOK_ALERTS_URL=https://api.example.com/alerts

# External Services
VAULT_URL=https://vault.example.com
VAULT_TOKEN=vault_token
ELASTICSEARCH_URL=http://elasticsearch:9200
```

---

## 5) PRODUCTION API ENDPOINTS

### Authentication & Authorization
```typescript
// POST /api/v1/auth/login
{
  "username": "operator1",
  "password": "securepassword"
}
// Response: 200 { access_token, refresh_token, expires_in, user: {...} }

// POST /api/v1/auth/refresh
{
  "refresh_token": "..."
}

// POST /api/v1/auth/logout
// DELETE current refresh token
```

### DVR Management
```typescript
// GET /api/v1/dvrs?page=1&limit=10&status=online&search=site1
// Response: { dvrs: [...], pagination: {...}, filters: {...} }

// POST /api/v1/dvrs
{
  "name": "Site-1 Main DVR",
  "description": "Primary surveillance DVR for Site 1",
  "brand": "Hikvision",
  "model": "DS-7608NI-K2/8P",
  "ip": "192.168.1.100",
  "port": 554,
  "username": "admin",
  "password": "admin123",
  "rtsp_paths": {
    "channel_1": "/Streaming/Channels/101",
    "channel_2": "/Streaming/Channels/201"
  },
  "onvif_port": 80,
  "location": {
    "name": "Building A - Main Entrance",
    "coordinates": {"lat": 40.7128, "lng": -74.0060}
  },
  "timezone": "America/New_York"
}

// PUT /api/v1/dvrs/:id/test-connection
// Test ONVIF and RTSP connectivity

// POST /api/v1/dvrs/:id/discover-streams
// Auto-discover available RTSP streams
```

### Stream Management
```typescript
// POST /api/v1/streams
{
  "dvr_id": "uuid",
  "channel_number": 1,
  "profiles": ["high", "medium", "low"], // Adaptive bitrate
  "auto_restart": true,
  "max_viewers": 50
}

// GET /api/v1/streams/:id/status
// Response: { status, uptime, viewers, quality_metrics, errors }

// POST /api/v1/streams/:id/restart
// Restart with exponential backoff

// DELETE /api/v1/streams/:id
// Graceful shutdown with cleanup
```

### Analytics & Monitoring
```typescript
// GET /api/v1/analytics/dashboard
// Real-time dashboard metrics

// GET /api/v1/analytics/streams/:id/metrics?from=2024-01-01&to=2024-01-31
// Historical stream analytics

// GET /api/v1/health
// System health check
{
  "status": "healthy",
  "timestamp": "2024-01-01T00:00:00Z",
  "checks": {
    "database": "healthy",
    "redis": "healthy", 
    "ffmpeg": "healthy",
    "disk_space": "healthy"
  },
  "metrics": {
    "active_streams": 15,
    "total_viewers": 143,
    "cpu_usage": 45.2,
    "memory_usage": 67.8
  }
}
```

---

## 6) PRODUCTION-GRADE FFMPEG CONFIGURATION

### Adaptive Bitrate Streaming
```typescript
const ADAPTIVE_PROFILES = {
  high: {
    resolution: '1920x1080',
    videoBitrate: '4000k',
    audioBitrate: '128k',
    framerate: 30,
    preset: 'medium'
  },
  medium: {
    resolution: '1280x720', 
    videoBitrate: '2000k',
    audioBitrate: '96k',
    framerate: 30,
    preset: 'fast'
  },
  low: {
    resolution: '854x480',
    videoBitrate: '800k',
    audioBitrate: '64k',
    framerate: 25,
    preset: 'veryfast'
  }
};

// Master playlist generation
const generateFFmpegCommand = (rtspUrl: string, outputDir: string) => [
  '-rtsp_transport', 'tcp',
  '-reconnect', '1',
  '-reconnect_streamed', '1', 
  '-reconnect_delay_max', '5',
  '-i', rtspUrl,
  
  // Video processing
  '-c:v', 'libx264',
  '-preset', 'fast',
  '-profile:v', 'main',
  '-level:v', '4.0',
  '-pix_fmt', 'yuv420p',
  '-g', '60', // 2 second GOP for 30fps
  '-sc_threshold', '0',
  '-force_key_frames', 'expr:gte(t,n_forced*2)',
  
  // Audio processing
  '-c:a', 'aac',
  '-b:a', '128k',
  '-ar', '48000',
  '-ac', '2',
  
  // HLS output
  '-f', 'hls',
  '-hls_time', '2',
  '-hls_list_size', '5',
  '-hls_delete_threshold', '1',
  '-hls_flags', 'delete_segments+discont_start+split_by_time',
  '-hls_segment_filename', `${outputDir}/segment_%03d.ts`,
  '-master_pl_name', 'master.m3u8',
  
  // Quality variants
  '-var_stream_map', 'v:0,a:0 v:1,a:1 v:2,a:2',
  '-hls_segment_filename', `${outputDir}/stream_%v/segment_%03d.ts`,
  `${outputDir}/stream_%v/index.m3u8`
];
```

### Hardware Acceleration (Production)
```typescript
const HARDWARE_ACCELERATION = {
  nvidia: {
    decoder: ['-hwaccel', 'cuda', '-hwaccel_output_format', 'cuda'],
    encoder: ['-c:v', 'h264_nvenc', '-preset', 'p4']
  },
  intel: {
    decoder: ['-hwaccel', 'qsv'],
    encoder: ['-c:v', 'h264_qsv', '-preset', 'medium']
  },
  amd: {
    decoder: ['-hwaccel', 'amf'],
    encoder: ['-c:v', 'h264_amf']
  }
};
```

---

## 7) SECURITY ENHANCEMENTS (PRODUCTION)

### Password Security
```typescript
// Use bcrypt for user passwords
import bcrypt from 'bcrypt';

// Use AES-256-GCM for DVR passwords with key rotation
import crypto from 'crypto';

class SecurityService {
  private currentKeyId: string;
  private keys: Map<string, Buffer>;

  async encryptDVRPassword(password: string): Promise<string> {
    const keyId = this.currentKeyId;
    const key = this.keys.get(keyId);
    const iv = crypto.randomBytes(12);
    const cipher = crypto.createCipher('aes-256-gcm', key);
    cipher.setAAD(Buffer.from(keyId));
    
    let encrypted = cipher.update(password, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    const authTag = cipher.getAuthTag().toString('hex');
    
    return `${keyId}:${iv.toString('hex')}:${encrypted}:${authTag}`;
  }

  // Implement key rotation
  async rotateEncryptionKeys(): Promise<void> {
    // Generate new key, re-encrypt all DVR passwords
  }
}
```

### Rate Limiting & DDoS Protection
```typescript
import { RateLimiterRedis } from 'rate-limiter-flexible';

const rateLimiters = {
  login: new RateLimiterRedis({
    storeClient: redisClient,
    keyPrefix: 'login_fail',
    points: 5, // Number of attempts
    duration: 900, // Per 15 minutes
    blockDuration: 900, // Block for 15 minutes
  }),
  
  api: new RateLimiterRedis({
    storeClient: redisClient,
    keyPrefix: 'api_calls',
    points: 1000, // requests
    duration: 3600, // per hour
  }),
  
  streaming: new RateLimiterRedis({
    storeClient: redisClient, 
    keyPrefix: 'stream_start',
    points: 10, // stream starts
    duration: 300, // per 5 minutes
  })
};
```

### Input Validation
```typescript
import Joi from 'joi';

const dvrSchema = Joi.object({
  name: Joi.string().min(1).max(200).required(),
  ip: Joi.string().ip().required(),
  port: Joi.number().integer().min(1).max(65535).default(554),
  username: Joi.string().min(1).max(255).required(),
  password: Joi.string().min(1).max(255).required(),
  rtsp_paths: Joi.object().pattern(
    Joi.string(), 
    Joi.string().regex(/^\/[A-Za-z0-9\/\-_]+$/)
  ),
});

const validateMiddleware = (schema: Joi.ObjectSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details
      });
    }
    next();
  };
};
```

---

## 8) MONITORING & ALERTING (PRODUCTION)

### Metrics Collection
```typescript
import client from 'prom-client';

const metrics = {
  streamCount: new client.Gauge({
    name: 'active_streams_total',
    help: 'Number of active streams'
  }),
  
  streamUptime: new client.Histogram({
    name: 'stream_uptime_seconds',
    help: 'Stream uptime in seconds',
    labelNames: ['dvr_id', 'stream_id']
  }),
  
  ffmpegRestarts: new client.Counter({
    name: 'ffmpeg_restarts_total',
    help: 'FFmpeg process restarts',
    labelNames: ['dvr_id', 'reason']
  }),
  
  apiRequests: new client.Counter({
    name: 'api_requests_total',
    help: 'API requests',
    labelNames: ['method', 'endpoint', 'status']
  })
};
```

### Health Monitoring
```typescript
class HealthMonitor {
  async checkStreamHealth(streamId: string): Promise<HealthStatus> {
    const stream = await Stream.findById(streamId);
    const checks = await Promise.allSettled([
      this.checkFFmpegProcess(stream.ffmpeg_pid),
      this.checkHLSPlaylist(stream.hls_playlist_url),
      this.checkDiskSpace(stream.output_directory),
      this.checkNetworkLatency(stream.dvr_id)
    ]);
    
    return this.aggregateHealthScore(checks);
  }
  
  private async checkHLSPlaylist(url: string): Promise<boolean> {
    try {
      const response = await fetch(url);
      const content = await response.text();
      return content.includes('#EXTM3U') && response.ok;
    } catch {
      return false;
    }
  }
}
```

### Alert System
```typescript
interface AlertRule {
  condition: (metrics: any) => boolean;
  severity: 'low' | 'medium' | 'high' | 'critical';
  cooldown: number; // seconds
}

const alertRules: AlertRule[] = [
  {
    condition: (m) => m.streamDowntime > 300, // 5 minutes
    severity: 'high',
    cooldown: 600
  },
  {
    condition: (m) => m.diskUsage > 0.9, // 90% full
    severity: 'critical', 
    cooldown: 300
  },
  {
    condition: (m) => m.errorRate > 0.05, // 5% error rate
    severity: 'medium',
    cooldown: 900
  }
];
```

---

## 9) FRONTEND PRODUCTION FEATURES

### Real-time Dashboard
```typescript
// WebSocket connection for real-time updates
const useLiveMetrics = () => {
  const [metrics, setMetrics] = useState<StreamMetrics[]>([]);
  
  useEffect(() => {
    const ws = new WebSocket(`${WS_URL}/metrics`);
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setMetrics(data.streams);
    };
    return () => ws.close();
  }, []);
  
  return metrics;
};

// Advanced HLS Player with adaptive quality
const AdaptiveHLSPlayer = ({ streamUrl }: { streamUrl: string }) => {
  const videoRef = useRef<HTMLVideoElement>(null);
  const hlsRef = useRef<Hls>();
  const [quality, setQuality] = useState<'auto' | number>('auto');
  
  useEffect(() => {
    if (Hls.isSupported() && videoRef.current) {
      const hls = new Hls({
        enableWorker: true,
        lowLatencyMode: true,
        backBufferLength: 90
      });
      
      hls.loadSource(streamUrl);
      hls.attachMedia(videoRef.current);
      
      hls.on(Hls.Events.MANIFEST_PARSED, () => {
        videoRef.current?.play();
      });
      
      hls.on(Hls.Events.ERROR, (event, data) => {
        if (data.fatal) {
          switch (data.type) {
            case Hls.ErrorTypes.NETWORK_ERROR:
              hls.startLoad();
              break;
            case Hls.ErrorTypes.MEDIA_ERROR:
              hls.recoverMediaError();
              break;
            default:
              hls.destroy();
              break;
          }
        }
      });
      
      hlsRef.current = hls;
    }
    
    return () => {
      hlsRef.current?.destroy();
    };
  }, [streamUrl]);
  
  return (
    <div className="relative">
      <video
        ref={videoRef}
        controls
        className="w-full h-auto"
        playsInline
        muted
      />
      <QualitySelector 
        hls={hlsRef.current}
        currentQuality={quality}
        onQualityChange={setQuality}
      />
    </div>
  );
};
```

### Multi-Stream Grid View
```typescript
const MultiStreamGrid = ({ streams }: { streams: Stream[] }) => {
  const [layout, setLayout] = useState<'1x1' | '2x2' | '3x3' | '4x4'>('2x2');
  const [focusedStream, setFocusedStream] = useState<string | null>(null);
  
  return (
    <div className="stream-grid" data-layout={layout}>
      {streams.map((stream) => (
        <div 
          key={stream.id}
          className={`stream-cell ${focusedStream === stream.id ? 'focused' : ''}`}
          onClick={() => setFocusedStream(stream.id)}
        >
          <AdaptiveHLSPlayer streamUrl={stream.hls_playlist_url} />
          <StreamOverlay stream={stream} />
        </div>
      ))}
    </div>
  );
};
```

---

## 10) DEPLOYMENT & INFRASTRUCTURE (PRODUCTION)

### Docker Configuration
```dockerfile
# backend/Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime
RUN apk add --no-cache ffmpeg
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
RUN npm run build

USER node
EXPOSE 8000
CMD ["npm", "start"]
```

### Kubernetes Deployment
```yaml
# infrastructure/kubernetes/manifests/backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dvr-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dvr-backend
  template:
    metadata:
      labels:
        app: dvr-backend
    spec:
      containers:
      - name: backend
        image: your-registry/dvr-backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: dvr-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: dvr-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /api/v1/health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/v1/health/ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: streams-storage
          mountPath: /data/streams
      volumes:
      - name: streams-storage
        persistentVolumeClaim:
          claimName: streams-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: dvr-backend-service
spec:
  selector:
    app: dvr-backend
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

### NGINX Configuration
```nginx
# infrastructure/nginx/nginx.conf
upstream backend {
    least_conn;
    server backend-1:8000;
    server backend-2:8000;
    server backend-3:8000;
}

server {
    listen 443 ssl http2;
    server_name dvr.yourdomain.com;
    
    ssl_certificate /etc/ssl/certs/dvr.crt;
    ssl_certificate_key /etc/ssl/private/dvr.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    
    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=streams:10m rate=1r/s;
    
    # API routes
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # HLS streaming with caching
    location /streams/ {
        limit_req zone=streams burst=5 nodelay;
        
        # Cache HLS segments
        location ~* \.(m3u8)$ {
            expires -1;
            add_header Cache-Control "no-cache, no-store, must-revalidate";
            proxy_pass http://backend;
        }
        
        location ~* \.(ts)$ {
            expires 1h;
            add_header Cache-Control "public, immutable";
            proxy_pass http://backend;
        }
    }
    
    # Frontend
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
        
        # Static assets caching
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

### Monitoring Stack
```yaml
# infrastructure/monitoring/prometheus/config.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'dvr-backend'
    static_configs:
      - targets: ['backend-service:9090']
    metrics_path: /metrics
    scrape_interval: 10s

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### Terraform Infrastructure
```hcl
# infrastructure/terraform/main.tf
provider "aws" {
  region = var.aws_region
}

# VPC and networking
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "dvr-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
}

# RDS PostgreSQL
resource "aws_db_instance" "main" {
  identifier = "dvr-db"
  
  engine         = "postgres"
  engine_version = "14.9"
  instance_class = "db.t3.large"
  
  allocated_storage     = 100
  max_allocated_storage = 1000
  storage_encrypted     = true
  
  db_name  = "dvr_production"
  username = var.db_username
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  skip_final_snapshot = false
  final_snapshot_identifier = "dvr-db-final-snapshot"
}

# ElastiCache Redis
resource "aws_elasticache_subnet_group" "main" {
  name       = "dvr-cache-subnet"
  subnet_ids = module.vpc.private_subnets
}

resource "aws_elasticache_replication_group" "main" {
  replication_group_id       = "dvr-redis"
  description                = "Redis cluster for DVR system"
  
  node_type                  = "cache.t3.micro"
  port                       = 6379
  parameter_group_name       = "default.redis7"
  
  num_cache_clusters         = 3
  automatic_failover_enabled = true
  multi_az_enabled          = true
  
  subnet_group_name = aws_elasticache_subnet_group.main.name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "dvr-cluster"
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight           = 1
  }
}

# EFS for shared storage
resource "aws_efs_file_system" "streams" {
  creation_token = "dvr-streams"
  
  performance_mode = "generalPurpose"
  throughput_mode  = "provisioned"
  provisioned_throughput_in_mibps = 100
  
  encrypted = true
  
  lifecycle_policy {
    transition_to_ia = "AFTER_30_DAYS"
  }
}
```

---

## 11) TESTING STRATEGY (PRODUCTION)

### Unit Tests
```typescript
// tests/unit/services/ffmpegManager.test.ts
import { FFmpegManager } from '../../../src/services/ffmpegManager';
import { spawn } from 'child_process';

jest.mock('child_process');

describe('FFmpegManager', () => {
  let manager: FFmpegManager;
  
  beforeEach(() => {
    manager = new FFmpegManager();
    (spawn as jest.Mock).mockClear();
  });
  
  it('should start stream with correct ffmpeg arguments', async () => {
    const mockProcess = {
      pid: 12345,
      on: jest.fn(),
      kill: jest.fn()
    };
    (spawn as jest.Mock).mockReturnValue(mockProcess);
    
    const result = await manager.startStream('dvr-1', 'rtsp://test');
    
    expect(spawn).toHaveBeenCalledWith('ffmpeg', expect.arrayContaining([
      '-rtsp_transport', 'tcp',
      '-i', 'rtsp://test'
    ]));
    expect(result.pid).toBe(12345);
  });
  
  it('should handle ffmpeg process crash', async () => {
    const mockProcess = {
      pid: 12345,
      on: jest.fn(),
      kill: jest.fn()
    };
    (spawn as jest.Mock).mockReturnValue(mockProcess);
    
    await manager.startStream('dvr-1', 'rtsp://test');
    
    // Simulate process crash
    const exitHandler = mockProcess.on.mock.calls.find(call => call[0] === 'exit')[1];
    exitHandler(1, 'SIGTERM');
    
    // Should trigger restart logic
    expect(manager.getStreamStatus('dvr-1')).toBe('restarting');
  });
});
```

### Integration Tests
```typescript
// tests/integration/api/streams.test.ts
import request from 'supertest';
import { app } from '../../../src/server';
import { testDb } from '../../helpers/database';

describe('Stream API', () => {
  let authToken: string;
  let dvrId: string;
  
  beforeAll(async () => {
    await testDb.setup();
    const loginResponse = await request(app)
      .post('/api/v1/auth/login')
      .send({ username: 'testuser', password: 'testpass' });
    authToken = loginResponse.body.access_token;
  });
  
  afterAll(async () => {
    await testDb.teardown();
  });
  
  it('should start stream successfully', async () => {
    // First create a DVR
    const dvrResponse = await request(app)
      .post('/api/v1/dvrs')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        name: 'Test DVR',
        ip: '192.168.1.100',
        username: 'admin',
        password: 'admin123'
      });
    
    dvrId = dvrResponse.body.id;
    
    // Start stream
    const response = await request(app)
      .post(`/api/v1/streams`)
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        dvr_id: dvrId,
        profiles: ['medium']
      });
    
    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('id');
    expect(response.body).toHaveProperty('hls_playlist_url');
    expect(response.body.status).toBe('starting');
  });
});
```

### End-to-End Tests
```typescript
// tests/e2e/streaming.test.ts
import { chromium, Browser, Page } from 'playwright';

describe('Streaming E2E', () => {
  let browser: Browser;
  let page: Page;
  
  beforeAll(async () => {
    browser = await chromium.launch();
    page = await browser.newPage();
  });
  
  afterAll(async () => {
    await browser.close();
  });
  
  it('should play HLS stream in browser', async () => {
    await page.goto('http://localhost:3000');
    
    // Login
    await page.fill('[data-testid=username]', 'testuser');
    await page.fill('[data-testid=password]', 'testpass');
    await page.click('[data-testid=login-button]');
    
    // Start stream
    await page.click('[data-testid=start-stream-button]');
    
    // Wait for video element to be ready
    await page.waitForSelector('video[src*=".m3u8"]');
    
    // Verify video is playing
    const isPlaying = await page.evaluate(() => {
      const video = document.querySelector('video') as HTMLVideoElement;
      return !video.paused && video.currentTime > 0 && video.readyState > 2;
    });
    
    expect(isPlaying).toBe(true);
  });
});
```

### Load Testing
```typescript
// tests/load/streaming.test.ts
import { check } from 'k6';
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '2m', target: 10 }, // Ramp up
    { duration: '5m', target: 50 }, // Stay at 50 users
    { duration: '2m', target: 0 },  // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'], // 95% of requests under 2s
    http_req_failed: ['rate<0.1'],     // Error rate under 10%
  },
};

export default function() {
  const response = http.get(`${__ENV.BASE_URL}/api/v1/streams/active`);
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
}
```

---

## 12) OPERATIONAL PROCEDURES

### Backup Strategy
```bash
#!/bin/bash
# scripts/backup.sh

# Database backup
pg_dump $DATABASE_URL | gzip > /backups/db_$(date +%Y%m%d_%H%M%S).sql.gz

# Stream metadata backup
tar -czf /backups/streams_metadata_$(date +%Y%m%d_%H%M%S).tar.gz /data/streams/*/index.m3u8

# Configuration backup
kubectl get configmaps,secrets -o yaml > /backups/k8s_config_$(date +%Y%m%d_%H%M%S).yaml

# Upload to S3
aws s3 sync /backups/ s3://dvr-backups/$(hostname)/
```

### Disaster Recovery
```yaml
# disaster-recovery/playbook.yml
- name: DVR System Disaster Recovery
  hosts: backup_cluster
  tasks:
    - name: Restore database from backup
      shell: |
        gunzip -c {{ backup_file }} | psql {{ database_url }}
    
    - name: Deploy application stack
      k8s:
        state: present
        definition: "{{ item }}"
      loop:
        - "{{ lookup('file', 'manifests/backend.yaml') | from_yaml_all | list }}"
        - "{{ lookup('file', 'manifests/frontend.yaml') | from_yaml_all | list }}"
    
    - name: Verify system health
      uri:
        url: "{{ app_url }}/api/v1/health"
        method: GET
      register: health_check
      until: health_check.status == 200
      retries: 10
      delay: 30
```

### Monitoring Runbook
```markdown
# Incident Response Runbook

## Stream Down Alert
1. Check DVR connectivity: `curl -I http://DVR_IP`
2. Verify ffmpeg process: `ps aux | grep ffmpeg`
3. Check disk space: `df -h /data/streams`
4. Review logs: `kubectl logs -f deployment/dvr-backend`
5. Restart stream: `curl -X POST /api/v1/streams/{id}/restart`

## High Error Rate Alert
1. Check application logs for error patterns
2. Verify database connectivity
3. Check network latency to DVRs
4. Review recent deployments
5. Scale up if CPU/memory constrained

## Database Connection Issues
1. Check PostgreSQL status: `systemctl status postgresql`
2. Review connection pool metrics
3. Check for long-running queries: `SELECT * FROM pg_stat_activity`
4. Verify network connectivity
5. Failover to read replica if necessary
```

---

## 13) SECURITY COMPLIANCE

### GDPR Compliance
```typescript
// Data retention policy
const DATA_RETENTION_POLICIES = {
  audit_logs: 365, // days
  user_sessions: 30,
  stream_analytics: 730,
  dvr_passwords: 'indefinite', // encrypted, business requirement
};

// Data anonymization
class GDPRService {
  async anonymizeUser(userId: string): Promise<void> {
    await db.transaction(async (trx) => {
      // Anonymize personal data
      await trx('users').where('id', userId).update({
        username: `anon_${crypto.randomUUID()}`,
        email: `deleted_${Date.now()}@example.com`,
        deleted_at: new Date()
      });
      
      // Keep audit logs but anonymize
      await trx('audit_logs').where('user_id', userId).update({
        user_id: null,
        anonymized: true
      });
    });
  }
}
```

### SOC 2 Compliance
```typescript
// Access logging for SOC 2
const auditMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const auditLog = {
    user_id: req.user?.id,
    action: `${req.method} ${req.path}`,
    resource_type: extractResourceType(req.path),
    resource_id: req.params.id,
    ip_address: req.ip,
    user_agent: req.get('User-Agent'),
    details: {
      query: req.query,
      body: sanitizeForAudit(req.body)
    }
  };
  
  // Log after response
  res.on('finish', () => {
    auditLog.status_code = res.statusCode;
    AuditLog.create(auditLog);
  });
  
  next();
};
```

---

## 14) FINAL DELIVERABLES CHECKLIST

### Code Quality Requirements
- [ ] TypeScript with strict mode enabled
- [ ] ESLint + Prettier configuration
- [ ] 90%+ test coverage (unit + integration)
- [ ] Security scanning with Snyk/OWASP
- [ ] Performance benchmarks documented
- [ ] API documentation (OpenAPI 3.0)
- [ ] Code review process established

### Infrastructure Requirements  
- [ ] Multi-environment setup (dev/staging/prod)
- [ ] Infrastructure as Code (Terraform/CloudFormation)
- [ ] Secrets management (Vault/AWS Secrets Manager)
- [ ] Monitoring and alerting configured
- [ ] Backup and disaster recovery tested
- [ ] SSL/TLS certificates automated (Let's Encrypt)
- [ ] CDN configuration for static assets

### Security Requirements
- [ ] Penetration testing completed
- [ ] OWASP Top 10 vulnerabilities addressed
- [ ] Rate limiting and DDoS protection
- [ ] Input validation and sanitization
- [ ] Secure password storage and rotation
- [ ] RBAC system implemented
- [ ] Audit logging for compliance

### Operational Requirements
- [ ] Runbooks for common incidents
- [ ] Performance monitoring dashboards
- [ ] Log aggregation and searching
- [ ] Automated deployment pipeline
- [ ] Rollback procedures documented
- [ ] Capacity planning guidelines
- [ ] SLA/SLO definitions

---

## PRODUCTION IMPLEMENTATION NOTES

**This enhanced prompt addresses the original MVP limitations:**

1. **Scalability**: Supports multiple DVRs, adaptive bitrate, horizontal scaling
2. **Security**: Production-grade authentication, encryption, input validation
3. **Reliability**: Auto-restart, health monitoring, disaster recovery
4. **Observability**: Comprehensive logging, metrics, alerting
5. **Compliance**: GDPR, SOC 2, audit trails
6. **Performance**: Caching, CDN, database optimization
7. **Operations**: Infrastructure as code, automated deployment

**Implementation Timeline: 8-12 weeks for full production system**
