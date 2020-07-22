[![Community Project header](https://github.com/newrelic/open-source-office/raw/master/examples/categories/images/Experimental.png)](https://github.com/newrelic/open-source-office/blob/master/examples/categories/index.md#experimental)

# Trend Micro Configuration

To integrate with New Relic, there are 3 ways to send Cloud One Workload Security(Deep Security) logs to fluentd.

（ Cloud One Workload Security 又はDeep SecurityとNew Relicが連携を行うには下記3種類の方法があります ）

 Pattern 1. DSA → DSA Local Syslog → Fluentd → New Relic
 
 Pattern 2. DSA → DSM → DSM Local Syslog → Fluentd → New Relic
 
 Pattern 3. DSA → DSM(WS) → Amazon SNS → New Relic

Abbreviation:

DSA - Deep Security Agent

DSM - Deep Security Manager 

WS - Workload Security ( former name: Deep Security as a Service )

#  Pattern 1. DSA → DSA Local Syslog → Fluentd → New Relic

 **1. Configuring rsyslog where DSA is installed.

（DSAがインストールされているサーバのrsyslogを設定）**
```
# touch /var/log/dsa_event.log
# chmod 600 /var/log/dsa_event.log
```

```
# vi /etc/rsyslog.conf
---
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# Do not output local6 to messsages
*.info;mail.none;authpriv.none;cron.none;local6.none     /var/log/messages

# Save DS events to ds.log
local6.*                                                /var/log/dsa_event.log
---
```
------

 **2. Restarting rsyslog to enable above changes
 
 （rsyslogを再起動させ、設定を反映させる）**
```
# systemctl restart rsyslog
# systemctl status rsyslog
      Active: active (running) 
# netstat -nau | grep 514
	   udp        0      0 0.0.0.0:514             0.0.0.0:*   
```   
---
**3. Configure log rotation to prevent log full.

（ログの肥大化を防ぐために、ログローテートの設定を行う）** 
```
# vi /etc/logrotate.d/dslog
*Please configure [maxsize] to meet your environment.
（「maxsize」については環境によって異なりますので、ここでは参考値とさせて頂きます）
---
/var/log/dsa_event.log {
    missingok
    rotate 1
    dateext
    dateformat _%Y%m%d
    compress
    copytruncate
    daily
#    maxsize 1000M
    maxsize 1K
}
---
```
```
# vi /var/log/dsa_event.log
*This is for testing purpose to verify logrotate works or not so that please fullfill [dsa_event.log] file with over 1kbyte size.
（ファイルサイズが1K以上になるように何か書き込む）

# /usr/sbin/logrotate /etc/logrotate.d/dslog -d
*Make sure [dsa_event.log] is rotated.
```
```
# vi /etc/logrotate.d/dslog
*Please configure [maxsize] to meet your environment.
（「maxsize」については環境によって異なりますので、ここでは参考値とさせて頂きます）
---
/var/log/dsa_event.log {
    missingok
    rotate 1
    dateext
    dateformat _%Y%m%d
    compress
    copytruncate
    daily
#Configure maximum log file size
    maxsize 1000M
#    maxsize 1K
}
---
```
---

 **4. Configuring Deep Security**

For English, please refer following link.

*Forward system events to a syslog or SIEM server

*Forward security events directly in real time from agent computers to a syslog or SIEM server

https://help.deepsecurity.trendmicro.com/10/0/siem-syslog-forwarding.html

For Japanese, please refer following link.

「システムイベントをSyslogサーバまたはSIEMサーバに転送する」

「AgentコンピュータからSyslogサーバまたはSIEMサーバにリアルタイムで直接セキュリティイベントを転送する」
を参照ください。

https://help.deepsecurity.trendmicro.com/10/0/ja-jp/siem-syslog-forwarding.html#Protection_modules_DSM

 **5. Configuring fluentd**

Please refer [ FluentD Configuration for Trend Micro ] instruction.

（ Fluentdの設定は [ FluentD Configuration for Trend Micro ]を参照ください ）

---
#  Pattern 2. DSA → DSM → DSM Local Syslog → Fluentd → New Relic

 **1. Configuring rsyslog where DSM is installed.
　Please refer procedure no1 - no3 from Pattern 1 for DSM rsyslog configuration.
（Pattern 1の手順１～３を参考にDSMにSyslogの設定を行う）**

 **2. Configuring Deep Security**

For English, please refer following link.

*Forward system events to a syslog or SIEM server

*Forward security events from the agent computers via the Deep Security Manager

https://help.deepsecurity.trendmicro.com/10/0/siem-syslog-forwarding.html

For Japanese, please refer following link.

「システムイベントをSyslogサーバまたはSIEMサーバに転送する」

「AgentコンピュータからDeep Security Manager経由でセキュリティイベントを転送する」
を参照ください。

https://help.deepsecurity.trendmicro.com/10/0/ja-jp/siem-syslog-forwarding.html#Protection_modules_DSM

 **3. Configuring fluentd**

Please refer [ FluentD Configuration for Trend Micro ] instruction.

（ Fluentdの設定は [ FluentD Configuration for Trend Micro ]を参照ください ）


---
#  Pattern 3. DSA → DSM(WS) → Amazon SNS → New Relic

 **1. Set up Amazon SNS and its integration.**

For English, please refer following link.

https://help.deepsecurity.trendmicro.com/event-sns.html


For Japanese,  please refer following link.

https://help.deepsecurity.trendmicro.com/ja-jp/event-sns.html


 **2. Configuring fluentd**

Please refer [ FluentD Configuration for Trend Micro ] instruction.

（ Fluentdの設定は [ FluentD Configuration for Trend Micro ]を参照ください ）

# FluentD Configuration for Trend micro
You can use FluentD configuration to store Trend micro logs to New Relic

Please see `fluentd.config` and check configuration.

Please replace string `{YOUR_INSIGHTS_INSERT_API_KEY}` to your insight insert API key then copy the configuration.

[Generate your insert API Key](https://docs.newrelic.com/docs/apis/get-started/intro-apis/types-new-relic-api-keys#event-insert-key)

## License
"FluentD Configuration for Trend micro" is licensed under the [Apache 2.0](http://apache.org/licenses/LICENSE-2.0.txt) License.
>[If applicable: The [Project Name] also uses source code from third party libraries. Full details on which libraries are used and the terms under which they are licensed can be found in the third party notices document.]
