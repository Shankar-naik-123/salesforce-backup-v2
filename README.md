# Salesforce Backup v2.0

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Java](https://img.shields.io/badge/Java-11%2B-blue.svg)](https://www.oracle.com/java/)
[![Salesforce API](https://img.shields.io/badge/Salesforce-Bulk%20API%20v2-00A1E0.svg)](https://developer.salesforce.com/docs/atlas.en-us.api_asynch.meta/api_asynch/)

A powerful command-line tool for backing up Salesforce data using the Bulk API v2. Backs up all queryable objects to CSV files with real-time progress tracking and robust error handling.

## Features

✅ **Complete Data Backup** - Backs up all queryable Salesforce objects to CSV files  
✅ **Bulk API v2** - Fast, efficient data extraction using Salesforce's latest API  
✅ **Parallel Processing** - Multi-threaded execution for maximum performance  
✅ **Real-time Progress** - Live progress updates with ETA and success/failure counters  
✅ **Smart Error Handling** - Gracefully skips unsupported objects and continues backup  
✅ **Configurable** - Backup all objects or specify which ones to include/exclude  
✅ **No Record Limits** - Full backup of all data, not just samples

## Requirements

- Java 11 or higher
- Valid Salesforce credentials (username, password, and security token)
- Internet connection

## Installation

1. **Download the latest release**
   ```bash
   # Clone the repository
   git clone https://github.com/YOUR_USERNAME/salesforce-backup-v2.git
   cd salesforce-backup-v2
   ```

2. **Build the project**
   ```bash
   mvn clean package
   ```

   This creates `target/backupforce-v2-1.0.0-SNAPSHOT-jar-with-dependencies.jar`

## Configuration

1. **Copy the configuration template**
   ```bash
   cp myconfig.properties.template myconfig.properties
   ```

2. **Edit `myconfig.properties` with your Salesforce credentials**
   ```properties
   # Your Salesforce username
   sf.username = your-email@company.com
   
   # Your password + security token concatenated
   # Example: if password is "Pass123" and token is "ABC123XYZ"
   # Then: Pass123ABC123XYZ
   sf.password = YourPasswordPlusSecurityToken
   
   # Server URL
   sf.serverurl = https://login.salesforce.com  # For production
   # sf.serverurl = https://test.salesforce.com  # For sandbox
   
   # Where to save backup files
   outputFolder = C:/backupForce2/backup
   
   # Which objects to backup (* = all)
   backup.objects = *
   
   # Objects to exclude (comma-separated)
   backup.objects.exclude = Attachment
   ```

### Getting Your Salesforce Security Token

If you don't have your security token:
1. Log in to Salesforce
2. Click your profile picture → Settings
3. My Personal Information → Reset My Security Token
4. Check your email for the new token
5. Append the token to your password in the config file

## Usage

Run the backup with:

```bash
java -jar target/backupforce-v2-1.0.0-SNAPSHOT-jar-with-dependencies.jar --config myconfig.properties
```

Or use the provided batch file (Windows):
```bash
run.bat
```

### Example Output

```
22:00:15.123 [main] INFO  BackupForce v2 - Using Bulk API v2
22:00:15.124 [main] INFO  Config file: myconfig.properties
22:00:15.125 [main] INFO  Authenticating to Salesforce...
22:00:16.456 [main] INFO  Authenticated successfully
22:00:16.457 [main] INFO  Retrieving list of all objects...
22:00:17.890 [main] INFO  Found 487 objects to backup
22:00:18.001 [pool-1-thread-1] INFO  Starting Bulk API v2 query for: Account
22:00:18.002 [pool-1-thread-2] INFO  Starting Bulk API v2 query for: Contact
...
22:00:45.123 [main] INFO  Progress: 10/487 objects (8 successful, 2 failed) - ETA: 325 seconds
22:01:15.456 [main] INFO  Progress: 20/487 objects (17 successful, 3 failed) - ETA: 298 seconds
...
22:15:42.789 [main] INFO  ============================================================
22:15:42.790 [main] INFO  Backup completed!
22:15:42.791 [main] INFO  Total objects: 487
22:15:42.792 [main] INFO  Successful: 465
22:15:42.793 [main] INFO  Failed: 22
22:15:42.794 [main] INFO  Total time: 942 seconds
22:15:42.795 [main] INFO  Output folder: C:/backupForce2/backup
22:15:42.796 [main] INFO  ============================================================
```

## Configuration Options

### Backup Specific Objects

To backup only specific objects, edit `myconfig.properties`:

```properties
# Backup only these objects
backup.objects = Account,Contact,Opportunity,Case,Lead

# Exclude specific objects
backup.objects.exclude = ActivityHistory,EmailStatus
```

### Output Location

Change where CSV files are saved:

```properties
outputFolder = D:/Salesforce/Backups
```

### API Version

The tool uses API version 59.0 by default. To change it, modify `Config.java`.

## Troubleshooting

### Authentication Failed
- Verify your username and password are correct
- Ensure you've appended your security token to your password
- Check if you're using the correct server URL (login vs test.salesforce.com)
- Verify your IP is not restricted in Salesforce security settings

### Some Objects Failed to Backup
This is normal! Some objects are not supported by the Bulk API:
- `KnowledgeArticle` - Use KnowledgeArticleVersion instead
- Objects with compound fields (e.g., Location with address data)
- Some system objects (e.g., ListViewChartInstance)

The tool will log these as failures but continue backing up other objects.

### Out of Memory
If backing up very large orgs, increase Java heap size:
```bash
java -Xmx4g -jar target/backupforce-v2-1.0.0-SNAPSHOT-jar-with-dependencies.jar --config myconfig.properties
```

## How It Works

1. **Authentication** - Connects to Salesforce using SOAP Partner API
2. **Object Discovery** - Retrieves list of all queryable objects via DescribeGlobal
3. **Field Discovery** - For each object, describes fields via REST API
4. **Query Creation** - Creates Bulk API v2 query jobs with all fields
5. **Parallel Processing** - Runs up to 10 objects concurrently
6. **Download** - Streams CSV results to local files
7. **Summary** - Reports success/failure statistics

## Project Structure

```
backupForce2/
├── src/main/java/com/backupforce/
│   ├── BackupRunner.java          # Main application entry point
│   ├── auth/
│   │   └── SalesforceAuth.java    # Authentication logic
│   ├── bulkv2/
│   │   └── BulkV2Client.java      # Bulk API v2 client
│   └── config/
│       └── Config.java            # Configuration handler
├── src/main/resources/
│   └── logback.xml                # Logging configuration
├── pom.xml                        # Maven dependencies
├── myconfig.properties.template   # Configuration template
└── run.bat                        # Windows batch script
```

## Dependencies

- **Salesforce Web Services Connector (WSC)** - SOAP API client
- **Apache HttpClient 5** - REST API calls
- **Google Gson** - JSON parsing
- **SLF4J + Logback** - Logging

## Security Notes

⚠️ **Important**: 
- Never commit `myconfig.properties` to version control
- Store credentials securely
- Consider using environment variables for CI/CD
- Restrict file system permissions on backup files

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - See [LICENSE](LICENSE) file for details

## Roadmap

- [ ] Desktop GUI application (JavaFX)
- [ ] Scheduled backups
- [ ] Incremental backups (only changes since last backup)
- [ ] Metadata backup (objects, fields, validation rules, etc.)
- [ ] Restore functionality
- [ ] Compression (ZIP output)

## Support

For issues, questions, or feature requests, please open an issue on GitHub.

---

**Note**: This tool is not officially affiliated with or supported by Salesforce.
