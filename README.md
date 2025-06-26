# Magento-Elastic-Search-crash-Restart
<?php
// CONFIGURATION
$esHost = 'http://localhost:9200';
$logFile = '/var/log/elasticsearch_monitor.log';
$lastRestartFile = '/tmp/es_last_restart.txt';
$restartCooldownMinutes = 15;  // Wait at least this long between restarts
$email = 'your@email.com';     // Optional: email alert
$slackWebhook = '';            // Optional: Slack webhook

// TIME SETUP
$now = time();
$nowFormatted = date('Y-m-d H:i:s');

// CHECK IF ELASTICSEARCH IS UP
$ch = curl_init($esHost);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_TIMEOUT, 5);
$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

// LOAD LAST RESTART TIME
$lastRestart = 0;
if (file_exists($lastRestartFile)) {
    $lastRestart = (int)file_get_contents($lastRestartFile);
}
$minutesSinceLastRestart = ($now - $lastRestart) / 60;

// FUNCTION: LOGGING
function logEvent($message, $logFile) {
    file_put_contents($logFile, "[" . date('Y-m-d H:i:s') . "] $message\n", FILE_APPEND);
}

// IF ELASTICSEARCH IS DOWN
if ($httpCode !== 200) {
    if ($minutesSinceLastRestart >= $restartCooldownMinutes) {
        // Restart Elasticsearch
        exec('sudo systemctl restart elasticsearch', $output, $resultCode);
        file_put_contents($lastRestartFile, $now);  // Update timestamp

        logEvent("Elasticsearch DOWN. Restarted. Code: $resultCode", $logFile);

        // Email Alert
        mail($email, 'Elasticsearch Restarted', "[$nowFormatted] Elasticsearch was down and restarted. Result: $resultCode");

        // Slack Alert
        if (!empty($slackWebhook)) {
            $payload = ['text' => "*Elasticsearch Restarted*\nTime: $nowFormatted\nResult Code: $resultCode"];
            $ch = curl_init($slackWebhook);
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
            curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_exec($ch);
            curl_close($ch);
        }
    } else {
        $wait = ceil($restartCooldownMinutes - $minutesSinceLastRestart);
        logEvent("Elasticsearch DOWN. Skipped restart – cooldown ($wait mins left).", $logFile);
    }
} else {
    logEvent("Elasticsearch OK", $logFile);
}
=====

🔧 Elasticsearch Expert for Magento Linux Server – Urgent Stability Fix (No New Server)

Project Description:

Our Magento website (PHP, Linux server, Elasticsearch) was running fine up until yesterday, when we experienced a major server issue and had to rebuild from a snapshot.

Since the rebuild:

    Elasticsearch crashes every few hours (used to be once every month or two).

    I have a script that restarts Elasticsearch hourly if it's down.

    A freelancer made some tweaks, and it's better—but still crashes after a few hours.

    They suggest more RAM, but upgrading the server is not viable—we’re moving to Shopify later this year.

Prior to yesterday, performance was acceptable, and I’m simply looking to restore that same level of stability without unnecessary infrastructure upgrades.
✅ What I Need:

    Elasticsearch expert familiar with Magento setups on Linux.

    Review server logs and Elasticsearch configs.

    Identify why Elasticsearch is crashing so often now.

    Apply any efficient config/system tweaks to restore pre-crash reliability.

    No server upgrade or RAM increase—fix within current system limits.

📌 Server Setup:

    Linux (Ubuntu/Debian)

    PHP Magento store

    Elasticsearch running as service

    Elasticsearch auto-restart script in place

❗ Important:

    No server upgrade required – temporary stability fix only until our Shopify migration is complete.

    You must know Elasticsearch tuning, JVM heap settings, and Linux server diagnostics.

    Must be available immediately and provide a quick fix ideally within 1 day.

🕐 Timeline:

ASAP – this is impacting our live website.
💬 When applying, please include:

    Your experience with Elasticsearch + Magento on Linux.

    A brief note on how you’d troubleshoot crashing Elasticsearch.

Let me know if you want me to add some of your server logs or current elasticsearch.yml or JVM settings into the job post to attract even more qualified candidates.

Would you also like a PHP monitoring script added for Elasticsearch status that sends Slack/email alerts in real time?
-------

🔧 Elasticsearch Expert for Magento Linux Server – Urgent Stability Fix (No New Server)

Project Description:

Our Magento website is hosted on a Linux server using Elasticsearch. Things were running okay (with minor issues) until yesterday, when a major server issue forced us to rebuild from a snapshot.

Before the rebuild:

    Elasticsearch would crash once every month or two — tolerable.

    I had a PHP script running hourly to restart it if down.

Since the rebuild:

    Crashing every 1–3 hours.

    Another freelancer tried optimizing, and now it lasts a bit longer, but still crashes multiple times a day.

    They recommend adding RAM, but that’s not viable — we’ll be migrating to Shopify later this year.

I just want Elasticsearch stable enough to last a few months without crashing every few hours.
✅ What I Need:

    Elasticsearch/Linux/Magento expert to:

        Investigate what changed post-rebuild.

        Review system logs, elasticsearch.yml, jvm.options, etc.

        Fix memory/config/runtime issues causing instability.

        Adjust JVM heap if necessary.

        Ensure Elasticsearch stays up — without major hardware upgrades.

    Optional: Review/enhance existing PHP script to monitor Elasticsearch (or suggest a better tool for this like Cron/Monit).

📌 Tech Stack:

    Linux (Ubuntu/Debian)

    PHP 7.x / Magento

    Elasticsearch (version 7.x)

    Cron + PHP watchdog script

    Apache/Nginx

🔐 Sample Monitoring Script Used (PHP):

<?php
$esHost = 'http://localhost:9200';
$ch = curl_init($esHost);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_TIMEOUT, 5);
$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

if ($httpCode !== 200) {
    // Restart Elasticsearch
    exec('sudo systemctl restart elasticsearch');

    // Optional: Email or Slack alert
    mail('you@example.com', 'Elasticsearch Restarted', 'It was down and has been restarted.');
}
?>

(Cron runs this hourly — needs improvement)
❗ Notes:

    No new server or RAM — just temporary stability until Shopify transition.

    Must be very experienced with Elasticsearch memory tuning and logs (dmesg, journalctl, etc.).

    Should provide clear documentation of what was changed.

🕐 Timeline:

ASAP – we’re live and can’t afford multi-hour outages.
💬 To Apply:

    Share examples of similar Elasticsearch fixes (esp. on Magento/Linux).

    Tell me how you’d troubleshoot this kind of crash.

    Let me know if you’d suggest replacing the PHP script with something like monit, systemd watchdog, etc.

