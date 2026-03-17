Using this plugin yuo will be able to send notification alert through email along with nagiosgraph.

# Installation Procedure

# Install required plugin (You required epel-release as well as crb repository)
[#] dnf install -y perl-Getopt-Long perl-Mail-Sendmail perl-Digest-MD5 perl-MIME-Base64 perl-File-Temp

# Add below section at the end of the command.cfg file.
[#] vim /usr/local/nagios/et/object/command.cfg
define command{
    command_name host-email-graph
    command_line /usr/local/nagios/libexec/nagios_send_host_mail.pl \
        -p "Company Name, Address" \
        -H "10.0.14.40:25" \
        -r "$CONTACTEMAIL$" \
        -f graph \
        -u
}
define command{
    command_name service-email-graph
    command_line /usr/local/nagios/libexec/nagios_send_service_mail.pl \
        -p "Company Name, Address" \
        -H "10.0.14.40:25" \
        -r "$CONTACTEMAIL$" \
        -f graph \
        -u
}

# Modify contacts.cfg file as like below file
[#] vim /usr/local/nagios/et/object/contacts.cfg
define contact{
        contact_name                    **sysadmin**
        use                             generic-contact
        alias                           System Admin
        email                           sysadmin@example.com
        service_notification_commands   service-email-graph
        host_notification_commands      host-email-graph
        }

define contact{
        contact_name                    zakir
        use                             generic-contact
        alias                           Md. Zakir Hossain
        email                           zhossain@example.com
        service_notification_commands   service-email-graph
        host_notification_commands      host-email-graph
        }

define contactgroup {
		contactgroup_name             	sysadmin
		alias                         	System Administrators
		members                       	**sysadmin**, zakir
}

# Modify templates.cfg file as like below file
[#] vim /usr/local/nagios/et/object/templates.cfg
define service{
        name                            important-service       
        use                             generic-service         
        max_check_attempts              3                       
        check_interval                  1                       
        retry_interval                  1                       
        notifications_enabled           1
        check_period                    24x7
        contact_groups                  **sysadmin**                
        notification_options            w,u,c,r                 
        notification_interval           60                      
        notification_period             24x7                    
        register                        0
        }
# Modify linux-services.cfg file as like below file
[#] vim /usr/local/nagios/et/object/linux-services.cfg
#Below service must by available otherwise nagios_send_host_mail.pl will not work.
#check-host-alive service must be active if you want to send host graph with email notification

define service {
    use                     important-service,nagiosgraph
    host_name               ns01,ns02
    service_description     check-host-alive
    check_command           check-host-alive
    contact_groups          sysadmin
}

# Current Load
define service {
    use                     important-service,nagiosgraph
    host_name               ns01,ns02
    service_description     Current Load
    check_command           check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
    contact_groups          sysadmin
}

** Note: Add all the services that you want to monitor and get the rich-text based alert along with nagiosgraph.
        
# Enable Environment Macros
[#] vim /usr/local/nagios/etc/nagios.cfg
enable_environment_macros=1

# Modify nagios_send_host_mail.pl & nagios_send_service_mail.pl as like below
[#] vim /usr/local/nagios/libexec/nagios_send_host_mail.pl
my $mail_sender        = "Nagios Monitoring <nagios\@example.com>";
my $nagios_cgiurl      = "http://nagios.example.com/nagios/cgi-bin";
my $test_host          = "localhost";    
my $test_service       = "Current Load"; 
my $ngraph_cgiurl      = "http://nagios.example.com/nagios/cgi-bin/show.cgi";
my $rrd_basedir        = "/usr/local/nagiosgraph/var/rrd/";
my $o_smtphost         = "10.0.14.40";
my $domain             = "\@example.com"; 
my $logofile = "/var/www/html/fm4dd/images/logo.png";
**Note: For this kind of SMTP configuration You required SMTP realy without authentication

# Create logo directory for Rich-Text based notification
mkdir -p /var/www/html/fm4dd/images/
copy logo.png into /var/www/html/fm4dd/images/ this directory

Finally reload the nagios service and test the alert

[#] perl /usr/local/nagios/libexec/nagios_send_service_mail.pl -t -H "10.0.14.40:25"  -p "ITC PLC, Tejgoan Branch" -r "zakirpcs@gmail.com" -f graph -u
[#] perl /usr/local/nagios/libexec/nagios_send_host_mail.pl -t -H "10.0.14.40:25"  -p "ITC PLC, Tejgoan Branch" -r "zakirpcs@gmail.com" -f graph -u

Usage: /usr/local/nagios/libexec/nagios_send_service_mail.pl [-v] [-V] [-h] [-t] [-H <SMTP host>] [-p <customername>]
       [-r <to_recipients> or -g <to_group>] [-c <cc_recipients>] [-b <bcc_recipients>]
       [-f <text|html|multi|graph>] [-u] [-l <en|jp|fr|de|es|(or other languages if added)>]


Now check your email you will get email as like below:
<img width="315" height="346" alt="nagiosgraph-alert" src="https://github.com/user-attachments/assets/df11f20c-c1f2-46c9-821c-5d3f802634e3" />

