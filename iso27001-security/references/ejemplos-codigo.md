# Ejemplos de codigo: ISO 27001 para RSM Peru

Estos ejemplos muestran la implementacion concreta de los controles definidos en el skill principal.

## Control de acceso con Azure AD

```csharp
// Program.cs - Configuracion de autorizacion
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAuditorClaim", policy =>
        policy.RequireClaim("role", "Auditor"));
});

// Controller protegido
[Authorize(Roles = "Auditor,Admin")]
public class AuditController : ControllerBase
{
    [Authorize(Policy = "RequireAuditorClaim")]
    public async Task<IActionResult> GetAuditLogs()
    {
        // Solo auditores pueden ver logs
    }
}
```

## Middleware de auditoria

```csharp
public class AuditMiddleware
{
    private readonly RequestDelegate _next;

    public AuditMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, AuditService auditService)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var ipAddress = context.Connection.RemoteIpAddress?.ToString();
        var action = $"{context.Request.Method} {context.Request.Path}";

        await auditService.LogAsync(new AuditLog
        {
            UserId = userId ?? "Anonymous",
            Action = action,
            Timestamp = DateTime.UtcNow,
            IpAddress = ipAddress
        });

        await _next(context);
    }
}
```

## Modelo AuditLog

```csharp
public class AuditLog
{
    public int Id { get; set; }
    public string UserId { get; set; }
    public string Action { get; set; }
    public string EntityType { get; set; }
    public string EntityId { get; set; }
    public DateTime Timestamp { get; set; }
    public string IpAddress { get; set; }
    public string Changes { get; set; }  // JSON
}
```

## Tabla SQL con retencion de 7 anos

```sql
CREATE TABLE AuditLogs (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    UserId NVARCHAR(450) NOT NULL,
    Action NVARCHAR(500) NOT NULL,
    EntityType NVARCHAR(256),
    EntityId NVARCHAR(256),
    Timestamp DATETIME2 NOT NULL,
    IpAddress NVARCHAR(45),
    Changes NVARCHAR(MAX),
    RetentionDate DATETIME2 NOT NULL DEFAULT DATEADD(YEAR, 7, GETUTCDATE())
);

CREATE PROCEDURE sp_ArchiveOldAuditLogs
AS
BEGIN
    INSERT INTO AuditLogsArchive
    SELECT * FROM AuditLogs
    WHERE RetentionDate < GETUTCDATE();

    DELETE FROM AuditLogs
    WHERE RetentionDate < GETUTCDATE();
END;
```

## Data Protection Service

```csharp
public class DataProtectionService
{
    private readonly IDataProtector _protector;

    public DataProtectionService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("RSMPeru.DataProtection");
    }

    public string Encrypt(string plainText) => _protector.Protect(plainText);
    public string Decrypt(string cipherText) => _protector.Unprotect(cipherText);
}
```

## Configuracion JWT Bearer

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ValidateIssuer = true,
            ValidIssuer = "rsmperu-api",
            ValidateAudience = true,
            ValidAudience = "rsmperu-app",
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };
    });

var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(claims),
    Expires = DateTime.UtcNow.AddHours(8),
    Issuer = "rsmperu-api",
    Audience = "rsmperu-app",
    SigningCredentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
};
```
