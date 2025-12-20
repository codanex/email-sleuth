# Email Sleuth

A Rust application to discover and verify professional email addresses based on contact names and company websites. This tool helps you find valid email addresses for business contacts when you have their name and company domain.

## Installation and Setup

### One-Line Unified Installation

Our unified installer handles everything you need:

```bash
curl -fsSL https://raw.githubusercontent.com/buyukakyuz/email-sleuth/main/setup.sh | bash
```

This will:
1. Install the Email Sleuth binary
2. Install ChromeDriver for enhanced verification (if desired)
3. Set up service management scripts
4. Create a default configuration

### Verify Installation

```bash
# Check if email-sleuth is properly installed
es --version
```

### Alternative Installation Methods

#### Manual Binary Download (All Platforms)

If you prefer not to use the script, or are on Windows:

1.  Go to the [**Releases page**](https://github.com/buyukakyuz/email-sleuth/releases).
2.  Find the latest release and locate the correct archive under "Assets" for your Operating System and Architecture.
3.  Download and extract the archive.
4.  Move the extracted `email-sleuth` (or `email-sleuth.exe` on Windows) executable to a directory in your system's `PATH`.
5.  **(Linux/macOS only)** Make the binary executable: `chmod +x /path/to/email-sleuth`.
6.  Verify by opening a **new** terminal and running: `email-sleuth --version`.

#### Build from Source (Developers)

Requires the Rust toolchain (>= 1.70 recommended).

```bash
# 1. Install Rust: https://www.rust-lang.org/tools/install
# 2. Clone the repository
git clone https://github.com/buyukakyuz/email-sleuth.git
cd email-sleuth

# 3. Build the optimized release binary
cargo build --release

# 4. The executable is at target/release/email-sleuth (or .exe)
#    Copy it to your PATH
```

## Usage

### Simple Single Contact Lookup

```bash
# Basic search (SMTP verification only)
es "John Doe" example.com

# Enhanced search (using API checks)
es -m enhanced "John Doe" example.com

# Comprehensive search (using all verification methods)
es -m comprehensive "John Doe" example.com
```

### Batch Processing

```bash
# Process all contacts in input.json with basic verification
es -i contacts.json -o results.json

# Process with comprehensive verification
es -m comprehensive -i contacts.json -o results.json
```

### Managing ChromeDriver Service

```bash
# Check status
es --service status

# Start the service
es --service start

# Stop the service
es --service stop

# Restart the service
es --service restart
```

## Verification Modes

Email Sleuth offers three verification modes to balance speed, accuracy, and resource usage:

| Mode | Description | Methods Used | Best For |
|------|-------------|--------------|----------|
| **basic** | SMTP verification only | DNS, SMTP | Quick checks, most reliable when port 25 is open |
| **enhanced** | Adds API-based checks | DNS, SMTP, API | Better accuracy, works when SMTP is partially blocked |
| **comprehensive** | Full verification suite | DNS, SMTP, API, Headless Browser | Highest accuracy, especially for major email providers |

## Input File Format (`input.json`) for Batch Mode

A JSON array of objects. Each object needs name fields (`first_name` and `last_name`) and a `domain`.

```json
[
  {
    "first_name": "John",
    "last_name": "Smith",
    "domain": "example.com"
  },
  {
    "first_name": "Jane",
    "last_name": "Doe",
    "domain": "acme.com"
  }
]
```
*(See `examples/example-contacts.json` for a more detailed example)*

## Output Format (`results.json`)

The tool produces a detailed JSON output for each contact processed. In CLI mode with `--stdout true`, a simplified summary is printed. When outputting to a file, the full structure is saved.

```json
[
  {
    "contact_input": { /* Original input contact data */ },
    "email": "john.smith@example.com", // Best guess found (or null)
    "confidence_score": 8,             // Confidence (0-10) for 'email'
    "found_emails": [                  // All plausible candidates found
      {
        "email": "john.smith@example.com",
        "confidence": 8,
        "source": "pattern", // "pattern", "scraped", "smtp", "api", "headless", etc.
        "is_generic": false,
        "verification_status": true, // true (exists), false (doesn't), null (inconclusive/skipped)
        "verification_message": "SMTP Verification OK: 250 2.1.5 Ok"
      }
      // ... other candidates
    ],
    "methods_used": ["pattern_generation", "smtp_verification"], // Methods used during discovery
    "verification_log": { /* Detailed verification check logs */ },
    "email_finding_skipped": false, // True if input was invalid
    "email_finding_error": null   // Unexpected processing errors
  },
  // ... results for other contacts
]
```

## Configuration

Your configuration file is located at `~/.config/email-sleuth/config.toml` after installation with the setup script.

## SMTP Requirements

Email verification using SMTP requires outbound access to port 25, which many ISPs block. If you see "Connection timed out" or similar errors, try:

1. Using the `enhanced` or `comprehensive` modes which include alternative verification methods
2. Running on a cloud server (AWS EC2, DigitalOcean, etc.)
3. Using a VPN service that allows port 25 traffic

The tool will automatically test your SMTP connectivity during startup and warn you if it's blocked.

## Troubleshooting

### SMTP Connection Issues

If you see errors about SMTP connectivity:

1. Your ISP may be blocking port 25
2. Try using the enhanced verification modes:
   ```bash
   es -m enhanced "John Smith" acme.com
   ```

## License

MIT License
