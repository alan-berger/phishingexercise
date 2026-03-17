# Phishing Exercise Tool

A PHP-based tool designed to help IT departments conduct controlled phishing awareness exercises by sending simulated LinkedIn invitation emails to employees.

## Purpose

This tool enables IT security teams to:
- Test employee awareness of phishing attacks
- Track which users click on suspicious links
- Conduct targeted security awareness training exercises
- Identify employees who may need additional security training

## How It Works

The script reads employee data from a CSV file and generates personalised fake LinkedIn invitation emails. Each email contains:
- Personalised greeting using the employee's first and last name
- Realistic LinkedIn invitation format
- Tracking links that are logged by the web server

When an employee clicks any link in the email, their interaction is logged via a unique tracking hash, allowing IT departments to identify which employees fell for the simulated phishing attempt.

## Features

- **Personalised emails**: Each message addresses the recipient by name
- **Tracking mechanism**: Uses RIPEMD-160 hashing of email addresses to create unique tracking identifiers
- **Realistic design**: Mimics actual LinkedIn invitation emails, although any HTML email can be used, it doesn't need to be LinkedIn
- **Safe testing**: Includes demonstration mode (emails are echoed rather than sent by default)

## Requirements

- PHP 5.3 or higher
- Access to PHP `mail()` function (for actual sending)
- Web server with PHP support
- CSV file containing employee data

## CSV Format

The input CSV file (`userlist.csv`) should be formatted as follows:

```
firstname,lastname,emailaddress
John,Smith,john.smith@company.com
Jane,Doe,jane.doe@company.com
```

**Important**: No header row should be included in the CSV file.

## Installation

1. Upload `phishingexercise.php` to your web server
2. Create a `userlist.csv` file in the same directory
3. Ensure the web server has read permissions for the CSV file
4. (Optional) Set up a tracking endpoint at `https://www.example.org/tracking.php`

## Configuration

Before using the tool, update the following variables in the code:

### Email Headers
```php
$mailheaders = "From: example@example.org \r\n";
$mailheaders .= "Reply-To: example@example.org \r\n";
```
Replace `example@example.org` with a suitable email address that targets the sender.

### Tracking URLs
Replace all instances of `https://www.example.org/tracking.php` with your actual tracking endpoint URL (if used)

### Sender Profile
Update the sender's name and details in the email template:
```php
<span>John Doe</span>
<p>Unknown</p>
<p>Generic</p>
```

## Usage

### Demonstration Mode (Default)

By default, the script runs in demonstration mode and outputs email details to the browser instead of sending them:

```bash
php phishingexercise.php
```

Or access via web browser:
```
http://yourserver.com/phishingexercise.php
```

The output will display:
- Recipient email address
- Email subject line
- Email headers
- Full HTML message content

### Sending Actual Emails

To send actual emails, uncomment the following line in the code:

```php
// mail($mailto, $mailsubject, $mailmessage, $mailheaders);
```

Change it to:

```php
mail($mailto, $mailsubject, $mailmessage, $mailheaders);
```

**Warning**: Ensure you have proper authorisation before sending phishing exercise emails to employees.

## Tracking Implementation

### How Tracking Works

1. Each user's email address is hashed using RIPEMD-160
2. The hash is appended to all links as a query parameter: `?ref=[hash]`
3. When a user clicks any link, the web server log will create an entry as the URL will not exist, or alternatively redirected to your tracking endpoint if configured
4. The tracking endpoint logs the hash and redirects the user appropriately

### Setting Up Tracking

(Optional) Create a `tracking.php` file on your server:

```php
<?php
// Log the tracking reference
$ref = $_GET['ref'] ?? 'unknown';
$timestamp = date('Y-m-d H:i:s');
$ip = $_SERVER['REMOTE_ADDR'];

// Log to file
file_put_contents('clicks.log', 
    "$timestamp | $ref | $ip\n", 
    FILE_APPEND
);

// Redirect to a safe page or show awareness message
header('Location: https://yourcompany.com/security-awareness');
exit;
?>
```

### Analysing Results

After the exercise, you can:
1. Review server logs to see which tracking hashes were accessed
2. Cross-reference hashes with your CSV file to identify users
3. Generate reports on click-through rates
4. Schedule follow-up training for users who clicked

## Security Considerations

### Legal and Ethical Requirements

- **Authorisation**: Obtain written approval from management before conducting exercises
- **HR Coordination**: Work with HR to ensure exercises comply with company policies
- **Employee Communication**: Consider informing employees that periodic security tests will occur (without specific timing)
- **Privacy**: Handle tracking data responsibly and in compliance with data protection regulations

### Best Practices

1. **Scope Control**: Only send to employees within your organisation
2. **Safe Landing Page**: Redirect clickers to an educational page, not a malicious-looking site
3. **No Credential Harvesting**: Never attempt to collect passwords or sensitive information
4. **Reasonable Timing**: Avoid sending during high-stress periods or outside business hours
5. **Follow-Up Training**: Provide constructive education, not punishment
6. **Data Retention**: Establish clear policies for how long tracking data is retained

### Technical Security

- Store the `userlist.csv` file outside the web root if possible
- Protect the CSV file with appropriate file permissions (600 or 640)
- Use HTTPS for all tracking URLs
- Implement rate limiting to prevent abuse
- Sanitise all output to prevent XSS vulnerabilities

## Customisation

### Email Template

The email uses a HTML template clone of LinkedIn's design. Although there is no limit to the amount of possible templates you could target, for example other social media organisations. You can customise:
- Sender name and profile information
- Profile picture URL (`johndoe.png`)
- LinkedIn logo URLs
- Button text and styling
- Footer content

### Tracking Hash Algorithm

The default uses RIPEMD-160. You can change to other algorithms:

```php
// SHA-256 (more common)
$tracker = hash('sha256', $csvemail);

// MD5 (less secure, but shorter)
$tracker = md5($csvemail);
```

## Troubleshooting

### Emails Not Sending

- Check that PHP `mail()` function is enabled on your server
- Verify SMTP configuration
- Check server mail logs for errors
- Ensure `From` address is valid and not blacklisted

### CSV Not Reading

- Verify file path is correct
- Check file permissions
- Ensure CSV format matches expected structure
- Look for BOM (Byte Order Mark) at file start

### Tracking Not Working

- Verify tracking URL is accessible
- Check web server logs for 404 errors
- Ensure tracking script has write permissions for log files
- Test tracking URL manually with sample hash

## Output Example

When run in demonstration mode, the script produces output similar to:

```
To: john.smith@company.com
Subject: John, please add me to your LinkedIn network
Headers:
From: example@example.org
Reply-To: example@example.org
MIME-Version: 1.0
Content-Type: text/html; charset=ISO-8859-1

Message:
[HTML content of the email displayed here]
```

## Limitations

- Requires properly configured PHP mail server
- May be flagged by spam filters
- Tracking relies on users having images/external content enabled
- Some email clients may block external images (LinkedIn logos)

## Legal Disclaimer

This tool is intended for authorised security awareness training only. Unauthorised use of this tool to send phishing emails may violate:
- Computer Fraud and Abuse Act (CFAA)
- CAN-SPAM Act
- GDPR and other privacy regulations
- Company policies and employment agreements

Always obtain proper authorisation and work with legal counsel before conducting phishing exercises.

## Future Enhancements

Potential improvements for this tool:
- Web-based interface for campaign management
- Multiple email templates
- Scheduled sending
- Automated reporting
- Integration with learning management systems
- A/B testing capabilities
- Multi-language support

## Support

For issues or questions about this tool, contact your IT security team.

## License

This tool is provided for educational and authorised security testing purposes only. Use at your own risk and in compliance with all applicable laws and regulations.
