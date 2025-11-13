# jp-go-config Package Usage

*Category: go*

## When to Use

Use jp-go-config when you need standardized configuration management with:

- Structured configuration types (Database, Redis, HTTP, etc.)
- Environment variable loading with Viper
- Validation of configuration values
- Test configuration builders
- Consistent configuration patterns across services

## Package Overview

jp-go-config provides predefined configuration structs and utilities that eliminate boilerplate and ensure consistent configuration across all services.

Repository: `github.com/JohnPlummer/jp-go-config`

## Import Convention

```go
import (
    goconfig "github.com/JohnPlummer/jp-go-config"
)
```

Use the `goconfig` alias for clarity and to avoid conflicts with local `config` packages.

## Configuration Types

### DatabaseConfig

Standard PostgreSQL configuration:

```go
type DatabaseConfig struct {
    Host              string
    Port              int
    User              string
    Password          string
    Database          string
    SSLMode           string
    MaxConns          int
    MinConns          int
    MaxConnLifetime   time.Duration
    MaxConnIdleTime   time.Duration
    HealthCheckPeriod time.Duration
}
```

Usage in main.go:

```go
func main() {
    cfg := &goconfig.DatabaseConfig{
        Host:     viper.GetString("database.host"),
        Port:     viper.GetInt("database.port"),
        User:     viper.GetString("database.user"),
        Password: viper.GetString("database.password"),
        Database: viper.GetString("database.name"),
        SSLMode:  viper.GetString("database.sslmode"),
        MaxConns: viper.GetInt("database.max_connections"),
    }

    db, err := pgxutils.NewConnection(cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
}
```

### RedisConfig

Standard Redis configuration:

```go
type RedisConfig struct {
    Host         string
    Port         int
    Password     string
    DB           int
    MaxRetries   int
    DialTimeout  time.Duration
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
}
```

### HTTPConfig

Standard HTTP server configuration:

```go
type HTTPConfig struct {
    Host         string
    Port         int
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
    IdleTimeout  time.Duration
}
```

Usage:

```go
cfg := &goconfig.HTTPConfig{
    Host:         viper.GetString("http.host"),
    Port:         viper.GetInt("http.port"),
    ReadTimeout:  viper.GetDuration("http.read_timeout"),
    WriteTimeout: viper.GetDuration("http.write_timeout"),
}

addr := fmt.Sprintf("%s:%d", cfg.Host, cfg.Port)
server := &http.Server{
    Addr:         addr,
    Handler:      handler,
    ReadTimeout:  cfg.ReadTimeout,
    WriteTimeout: cfg.WriteTimeout,
    IdleTimeout:  cfg.IdleTimeout,
}
```

## Environment Variables

Use Viper to load from environment variables:

```go
import "github.com/spf13/viper"

func init() {
    viper.AutomaticEnv()
    viper.SetEnvPrefix("APP")

    // Database defaults
    viper.SetDefault("database.host", "localhost")
    viper.SetDefault("database.port", 5432)
    viper.SetDefault("database.sslmode", "disable")
    viper.SetDefault("database.max_connections", 10)
}
```

Environment variables follow pattern: `APP_DATABASE_HOST`, `APP_DATABASE_PORT`

## Test Configuration Builders

Create test configurations without boilerplate:

```go
func NewTestDatabaseConfig(host string, port int) *goconfig.DatabaseConfig {
    return &goconfig.DatabaseConfig{
        Host:              host,
        Port:              port,
        User:              "testuser",
        Password:          "testpass",
        Database:          "testdb",
        SSLMode:           "disable",
        MaxConns:          10,
        MinConns:          2,
        MaxConnLifetime:   time.Hour,
        MaxConnIdleTime:   30 * time.Minute,
        HealthCheckPeriod: 1 * time.Minute,
    }
}
```

Usage in tests:

```go
var _ = BeforeSuite(func() {
    // Start testcontainer
    container, err := containers.StartSimplePostgreSQLContainer(ctx)
    Expect(err).NotTo(HaveOccurred())

    // Get container connection details
    host, _ := container.Host(ctx)
    port, _ := container.MappedPort(ctx, "5432")

    // Create test config
    testCfg := &goconfig.DatabaseConfig{
        Host:     host,
        Port:     port.Int(),
        User:     "testuser",
        Password: "testpass",
        Database: "testdb",
        SSLMode:  "disable",
        MaxConns: 10,
    }

    // Initialize database
    db, err = pgxutils.NewConnection(testCfg)
    Expect(err).NotTo(HaveOccurred())
})
```

## Configuration Validation

Validate configuration before use:

```go
func (c *DatabaseConfig) Validate() error {
    if c.Host == "" {
        return fmt.Errorf("database host is required")
    }
    if c.Port == 0 {
        return fmt.Errorf("database port is required")
    }
    if c.User == "" {
        return fmt.Errorf("database user is required")
    }
    if c.Database == "" {
        return fmt.Errorf("database name is required")
    }
    return nil
}
```

Usage:

```go
cfg := loadDatabaseConfig()
if err := cfg.Validate(); err != nil {
    log.Fatalf("invalid configuration: %v", err)
}
```

## Integration with jp-go-pgx-utils

DatabaseConfig is designed to work directly with jp-go-pgx-utils:

```go
import (
    goconfig "github.com/JohnPlummer/jp-go-config"
    pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"
)

func setupDatabase() (*pgxutils.Connection, error) {
    cfg := &goconfig.DatabaseConfig{
        Host:     viper.GetString("database.host"),
        Port:     viper.GetInt("database.port"),
        User:     viper.GetString("database.user"),
        Password: viper.GetString("database.password"),
        Database: viper.GetString("database.name"),
        SSLMode:  "require",
        MaxConns: 25,
    }

    return pgxutils.NewConnection(cfg)
}
```

## Configuration File Example

config.yaml:

```yaml
database:
  host: localhost
  port: 5432
  user: appuser
  password: ${DB_PASSWORD}  # From environment
  name: production_db
  sslmode: require
  max_connections: 25
  min_connections: 5

http:
  host: 0.0.0.0
  port: 8080
  read_timeout: 10s
  write_timeout: 10s
  idle_timeout: 120s

redis:
  host: localhost
  port: 6379
  password: ${REDIS_PASSWORD}
  db: 0
  max_retries: 3
```

## Loading Configuration

```go
func LoadConfig() error {
    viper.SetConfigName("config")
    viper.SetConfigType("yaml")
    viper.AddConfigPath(".")
    viper.AddConfigPath("./config")

    // Allow environment variables to override
    viper.AutomaticEnv()
    viper.SetEnvPrefix("APP")

    if err := viper.ReadInConfig(); err != nil {
        return fmt.Errorf("failed to read config: %w", err)
    }

    return nil
}
```

## Related Standards

- jp-go-pgx-utils.md - Database connection using DatabaseConfig
- configuration.md - Project-specific configuration patterns

## Common Pitfalls

1. **Don't create ad-hoc config structs** - Use jp-go-config types:

   ```go
   // Wrong
   type MyDatabaseConfig struct {
       Host string
       Port int
       ...
   }

   // Right
   cfg := &goconfig.DatabaseConfig{...}
   ```

2. **Always validate configuration** - Fail fast on startup:

   ```go
   if err := cfg.Validate(); err != nil {
       log.Fatalf("invalid config: %v", err)
   }
   ```

3. **Use environment variables for secrets** - Never commit passwords:

   ```yaml
   # Wrong
   password: my-secret-password

   # Right
   password: ${DB_PASSWORD}
   ```

4. **Set reasonable defaults** - Especially for test configurations:

   ```go
   viper.SetDefault("database.max_connections", 10)
   viper.SetDefault("database.sslmode", "require")
   ```
