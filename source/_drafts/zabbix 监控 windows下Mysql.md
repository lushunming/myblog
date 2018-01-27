---
layout: post
title: zabbix 监控 windows下Mysql
description: zabbix 监控 windows下Mysql
tagline:
image:
date: 2017/03/06
categories: [linux]
tags: [linux,zabbix,Mysql]
---

# zabbix 监控 windows下Mysql


## 环境
windows服务器

## 安装

1. 监控脚本

    mysql监控需要自己写脚本监控，在linux下写一个脚本很容易，但是windows下却很少。我是从别处搜索来的脚本。脚本主要是使用了mysqlAdmin命令来获取管理信息。
    * [MySQL_Ping.vbs](https://raw.githubusercontent.com/lushunming/zabbix_file/master/mysql/script/MySQL_Ping.vbs)是用来查看mysql是否存活
    ```
    Set objFS = CreateObject("Scripting.FileSystemObject")
    Set objArgs = WScript.Arguments
    str1 = getCommandOutput("E:\Program Files\MySQL\MySQL Server 5.5\bin\mysqladmin.exe -u用户 -p密码 --port=3306 ping")

    If Instr(str1,"alive") > 0 Then
    WScript.Echo 1
    Else
    WScript.Echo 0
    End If

    Function getCommandOutput(theCommand)

    Dim objShell, objCmdExec
    Set objShell = CreateObject("WScript.Shell")
    Set objCmdExec = objshell.exec(thecommand)
    getCommandOutput = objCmdExec.StdOut.ReadAll

    end Function
    ```
    其中的mysqlAdmin的路径需要自己修改，以及mysql用户和密码

    * [MySQL_Ext-Status_Script.vbs](https://raw.githubusercontent.com/lushunming/zabbix_file/master/mysql/script/MySQL_Ext-Status_Script.vbs)是获取一些额外的信息
    ```
     Set objFS = CreateObject("Scripting.FileSystemObject")
     Set objArgs = WScript.Arguments
     str1 = getCommandOutput("E:\Program Files\MySQL\MySQL Server 5.5\bin\mysqladmin.exe -uroot -pvortex --port=3306 extended-status")
     Arg = objArgs(0)
     str2 = Split(str1,"|")
     For i = LBound(str2) to UBound(str2)
     If Trim(str2(i)) = Arg Then
     WScript.Echo TRIM(str2(i+1))
     Exit For
     End If
     next

     Function getCommandOutput(theCommand)
     Dim objShell, objCmdExec
     Set objShell = CreateObject("WScript.Shell")
     Set objCmdExec = objshell.exec(thecommand)
     getCommandOutput = objCmdExec.StdOut.ReadAll
     end Function
    ```
     其中的mysqlAdmin的路径需要自己修改，以及mysql用户和密码

     * [Mysql_Version.vbs](https://raw.githubusercontent.com/lushunming/zabbix_file/master/mysql/script/Mysql_Version.vbs)文件是用来查询msyql的版本
     ```
     Set objFS = CreateObject("Scripting.FileSystemObject")
     Set objArgs = WScript.Arguments
     str1 = getCommandOutput("E:\Program Files\MySQL\MySQL Server 5.5\bin\mysql.exe -V")

     WScript.Echo str1

     Function getCommandOutput(theCommand)
     Dim objShell, objCmdExec
     Set objShell = CreateObject("WScript.Shell")
     Set objCmdExec = objshell.exec(thecommand)
     getCommandOutput = objCmdExec.StdOut.ReadAll
     end Function

     ```
    其中的mysqlAdmin的路径需要自己修改，以及mysql用户和密码

2. 配置

    修改`zabbix_agentd.win.conf`
    ```
    UnsafeUserParameters=1


    UserParameter=mysql.version,cscript /nologo  C:\zabbix\conf\Mysql_Version.vbs
    UserParameter=mysql.status[*],cscript /nologo  C:\zabbix\conf\MySQL_Ext-Status_Script.vbs $1
    UserParameter=mysql.ping,cscript /nologo  C:\zabbix\conf\MySql_Ping.vbs  
    ```
    * UnsafeUserParameters是允许参数中包含一个特殊字符，自定义脚本中往往都有，所以要设置为1
    * UserParameter增加监控项，我们是调用前面的脚本监控。
3. 导入模板

    导入监控模板
    ```
        <?xml version="1.0" encoding="UTF-8"?>
    <zabbix_export>
        <version>2.0</version>
        <date>2016-04-24T14:41:07Z</date>
        <groups>
            <group>
                <name>Templates</name>
            </group>
        </groups>
        <templates>
            <template>
                <template>Template App MySQL</template>
                <name>Template App MySQL</name>
                <description/>
                <groups>
                    <group>
                        <name>Templates</name>
                    </group>
                </groups>
                <applications>
                    <application>
                        <name>MySQL</name>
                    </application>
                </applications>
                <items>
                    <item>
                        <name>MySQL begin operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_begin]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL bytes received per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Bytes_received]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>Bps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>The number of bytes received from all clients. &#13;
    &#13;
    It requires user parameter mysql.status[*], which is defined in &#13;
    userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL bytes sent per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Bytes_sent]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>Bps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>The number of bytes sent to all clients.&#13;
    &#13;
    It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL commit operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_commit]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL delete operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_delete]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL insert operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_insert]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL queries per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Questions]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL rollback operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_rollback]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL select operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_select]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL slow queries</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Slow_queries]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>3</value_type>
                        <allowed_hosts/>
                        <units/>
                        <delta>0</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL status</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.ping</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>3</value_type>
                        <allowed_hosts/>
                        <units/>
                        <delta>0</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.ping, which is defined in userparameter_mysql.conf.&#13;
    &#13;
    0 - MySQL server is down&#13;
    1 - MySQL server is up</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap>
                            <name>Service state</name>
                        </valuemap>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL update operations per second</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Com_update]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>0</value_type>
                        <allowed_hosts/>
                        <units>qps</units>
                        <delta>1</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL uptime</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.status[Uptime]</key>
                        <delay>60</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>3</value_type>
                        <allowed_hosts/>
                        <units>uptime</units>
                        <delta>0</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.status[*], which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                    <item>
                        <name>MySQL version</name>
                        <type>0</type>
                        <snmp_community/>
                        <multiplier>0</multiplier>
                        <snmp_oid/>
                        <key>mysql.version</key>
                        <delay>3600</delay>
                        <history>7</history>
                        <trends>365</trends>
                        <status>0</status>
                        <value_type>1</value_type>
                        <allowed_hosts/>
                        <units/>
                        <delta>0</delta>
                        <snmpv3_contextname/>
                        <snmpv3_securityname/>
                        <snmpv3_securitylevel>0</snmpv3_securitylevel>
                        <snmpv3_authprotocol>0</snmpv3_authprotocol>
                        <snmpv3_authpassphrase/>
                        <snmpv3_privprotocol>0</snmpv3_privprotocol>
                        <snmpv3_privpassphrase/>
                        <formula>1</formula>
                        <delay_flex/>
                        <params/>
                        <ipmi_sensor/>
                        <data_type>0</data_type>
                        <authtype>0</authtype>
                        <username/>
                        <password/>
                        <publickey/>
                        <privatekey/>
                        <port/>
                        <description>It requires user parameter mysql.version, which is defined in userparameter_mysql.conf.</description>
                        <inventory_link>0</inventory_link>
                        <applications>
                            <application>
                                <name>MySQL</name>
                            </application>
                        </applications>
                        <valuemap/>
                        <logtimefmt/>
                    </item>
                </items>
                <discovery_rules/>
                <macros/>
                <templates/>
                <screens>
                    <screen>
                        <name>MySQL performance</name>
                        <hsize>2</hsize>
                        <vsize>1</vsize>
                        <screen_items>
                            <screen_item>
                                <resourcetype>0</resourcetype>
                                <width>500</width>
                                <height>200</height>
                                <x>0</x>
                                <y>0</y>
                                <colspan>1</colspan>
                                <rowspan>1</rowspan>
                                <elements>0</elements>
                                <valign>1</valign>
                                <halign>0</halign>
                                <style>0</style>
                                <url/>
                                <dynamic>0</dynamic>
                                <sort_triggers>0</sort_triggers>
                                <resource>
                                    <name>MySQL operations</name>
                                    <host>Template App MySQL</host>
                                </resource>
                                <max_columns>3</max_columns>
                                <application/>
                            </screen_item>
                            <screen_item>
                                <resourcetype>0</resourcetype>
                                <width>500</width>
                                <height>270</height>
                                <x>1</x>
                                <y>0</y>
                                <colspan>1</colspan>
                                <rowspan>1</rowspan>
                                <elements>0</elements>
                                <valign>1</valign>
                                <halign>0</halign>
                                <style>0</style>
                                <url/>
                                <dynamic>0</dynamic>
                                <sort_triggers>0</sort_triggers>
                                <resource>
                                    <name>MySQL bandwidth</name>
                                    <host>Template App MySQL</host>
                                </resource>
                                <max_columns>3</max_columns>
                                <application/>
                            </screen_item>
                        </screen_items>
                    </screen>
                </screens>
            </template>
        </templates>
        <triggers>
            <trigger>
                <expression>{Template App MySQL:mysql.ping.last(0)}=0</expression>
                <name>MySQL is down</name>
                <url/>
                <status>0</status>
                <priority>2</priority>
                <description/>
                <type>0</type>
                <dependencies/>
            </trigger>
        </triggers>
        <graphs>
            <graph>
                <name>MySQL bandwidth</name>
                <width>900</width>
                <height>200</height>
                <yaxismin>0.0000</yaxismin>
                <yaxismax>100.0000</yaxismax>
                <show_work_period>1</show_work_period>
                <show_triggers>1</show_triggers>
                <type>0</type>
                <show_legend>1</show_legend>
                <show_3d>0</show_3d>
                <percent_left>0.0000</percent_left>
                <percent_right>0.0000</percent_right>
                <ymin_type_1>0</ymin_type_1>
                <ymax_type_1>0</ymax_type_1>
                <ymin_item_1>0</ymin_item_1>
                <ymax_item_1>0</ymax_item_1>
                <graph_items>
                    <graph_item>
                        <sortorder>0</sortorder>
                        <drawtype>5</drawtype>
                        <color>00AA00</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Bytes_received]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>1</sortorder>
                        <drawtype>5</drawtype>
                        <color>3333FF</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Bytes_sent]</key>
                        </item>
                    </graph_item>
                </graph_items>
            </graph>
            <graph>
                <name>MySQL operations</name>
                <width>900</width>
                <height>200</height>
                <yaxismin>0.0000</yaxismin>
                <yaxismax>100.0000</yaxismax>
                <show_work_period>1</show_work_period>
                <show_triggers>1</show_triggers>
                <type>0</type>
                <show_legend>1</show_legend>
                <show_3d>0</show_3d>
                <percent_left>0.0000</percent_left>
                <percent_right>0.0000</percent_right>
                <ymin_type_1>0</ymin_type_1>
                <ymax_type_1>0</ymax_type_1>
                <ymin_item_1>0</ymin_item_1>
                <ymax_item_1>0</ymax_item_1>
                <graph_items>
                    <graph_item>
                        <sortorder>0</sortorder>
                        <drawtype>0</drawtype>
                        <color>C8C800</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_begin]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>1</sortorder>
                        <drawtype>0</drawtype>
                        <color>006400</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_commit]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>2</sortorder>
                        <drawtype>0</drawtype>
                        <color>C80000</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_delete]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>3</sortorder>
                        <drawtype>0</drawtype>
                        <color>0000EE</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_insert]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>4</sortorder>
                        <drawtype>0</drawtype>
                        <color>640000</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_rollback]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>5</sortorder>
                        <drawtype>0</drawtype>
                        <color>00C800</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_select]</key>
                        </item>
                    </graph_item>
                    <graph_item>
                        <sortorder>6</sortorder>
                        <drawtype>0</drawtype>
                        <color>C800C8</color>
                        <yaxisside>0</yaxisside>
                        <calc_fnc>2</calc_fnc>
                        <type>0</type>
                        <item>
                            <host>Template App MySQL</host>
                            <key>mysql.status[Com_update]</key>
                        </item>
                    </graph_item>
                </graph_items>
            </graph>
        </graphs>
    </zabbix_export>

    ```
    命名为你想要的名字，然后导入就可以了。

4. 监控
    新建一个主机，链接导入的模板就可以了。
