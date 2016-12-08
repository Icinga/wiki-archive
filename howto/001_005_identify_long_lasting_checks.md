#Identify long lasting checks

##Plugin

```
# cd /usr/lib/nagios/plugins
# wget http://wleibzon.bol.ucla.edu/nagios/plugins/profile_nagios_executiontime.pl -O profile_icinga_execution_time.pl
```
 
##Make it work with Icinga

Replace all occurences of Nagios with Icinga, if that tastes better. Plus edit the path to your status.dat - like i've done here already [Github Link](https://github.com/dnsmichi/icinga-plugins/blob/master/scripts/profile_icinga_execution_time.pl)
 
##Run it

```
# perl profile_icinga_execution_time.pl
......
Host: 2913localhost Service: 2913service Check Time: 0.020
Service: 2913service   Average Execution Time: 0.020 (sec)  NumChecks: 1
Service: 2534service   Average Execution Time: 0.021 (sec)  NumChecks: 1
Service: dep1   Average Execution Time: 0.023 (sec)  NumChecks: 1
Service: dep3   Average Execution Time: 0.023 (sec)  NumChecks: 1
Service: DISEÑOS   Average Execution Time: 0.023 (sec)  NumChecks: 1
Service: 1228ORACLESTANDBY   Average Execution Time: 0.023 (sec)  NumChecks: 1
Service: escalate1   Average Execution Time: 0.023 (sec)  NumChecks: 1
Service: 3088service1   Average Execution Time: 0.024 (sec)  NumChecks: 2
Service: StartupDelay   Average Execution Time: 0.024 (sec)  NumChecks: 1
Service: 3088service2   Average Execution Time: 0.025 (sec)  NumChecks: 2
Service: dep2   Average Execution Time: 0.025 (sec)  NumChecks: 1
Service: counter   Average Execution Time: 0.031 (sec)  NumChecks: 1
Service: 2291Macro Test   Average Execution Time: 0.034 (sec)  NumChecks: 1
Service: 2616ido2db Process   Average Execution Time: 0.043 (sec)  NumChecks: 1
Service: Service Temp+Hum   Average Execution Time: 0.104 (sec)  NumChecks: 1
Service: test_random_12   Average Execution Time: 0.113 (sec)  NumChecks: 6
Service: test_random_11   Average Execution Time: 0.113 (sec)  NumChecks: 9
Service: test_random_14   Average Execution Time: 0.113 (sec)  NumChecks: 13
Service: test_random_13   Average Execution Time: 0.114 (sec)  NumChecks: 6
Service: test_ok_19   Average Execution Time: 0.115 (sec)  NumChecks: 190
Service: test_ok_05   Average Execution Time: 0.115 (sec)  NumChecks: 190
Service: test_ok_16   Average Execution Time: 0.115 (sec)  NumChecks: 190
Service: test_ok_07   Average Execution Time: 0.115 (sec)  NumChecks: 197
Service: test_ok_18   Average Execution Time: 0.115 (sec)  NumChecks: 193
Service: test_ok_03   Average Execution Time: 0.115 (sec)  NumChecks: 184
Service: test_ok_14   Average Execution Time: 0.115 (sec)  NumChecks: 187
Service: test_ok_13   Average Execution Time: 0.116 (sec)  NumChecks: 194
Service: test_ok_10   Average Execution Time: 0.116 (sec)  NumChecks: 190
Service: test_ok_15   Average Execution Time: 0.116 (sec)  NumChecks: 189
Service: test_ok_17   Average Execution Time: 0.116 (sec)  NumChecks: 180
Service: test_ok_04   Average Execution Time: 0.116 (sec)  NumChecks: 192
Service: test_ok_08   Average Execution Time: 0.116 (sec)  NumChecks: 187
Service: test_ok_01   Average Execution Time: 0.116 (sec)  NumChecks: 191
Service: test_ok_09   Average Execution Time: 0.116 (sec)  NumChecks: 188
Service: test_ok_00   Average Execution Time: 0.116 (sec)  NumChecks: 192
Service: test_random_02   Average Execution Time: 0.116 (sec)  NumChecks: 7
Service: test_ok_11   Average Execution Time: 0.116 (sec)  NumChecks: 191
Service: test_ok_12   Average Execution Time: 0.116 (sec)  NumChecks: 194
Service: test_ok_06   Average Execution Time: 0.116 (sec)  NumChecks: 188
Service: test_ok_02   Average Execution Time: 0.117 (sec)  NumChecks: 193
Service: _all_   Average Execution Time: 0.197 (sec)  NumChecks: 4019
Service: test_random_17   Average Execution Time: 0.661 (sec)  NumChecks: 20
Service: test_random_03   Average Execution Time: 0.678 (sec)  NumChecks: 16
Service: test_random_15   Average Execution Time: 0.929 (sec)  NumChecks: 11
Service: test_random_08   Average Execution Time: 1.116 (sec)  NumChecks: 13
Service: test_random_00   Average Execution Time: 1.117 (sec)  NumChecks: 8
Service: test_random_04   Average Execution Time: 1.237 (sec)  NumChecks: 8
Service: test_random_19   Average Execution Time: 1.312 (sec)  NumChecks: 10
Service: test_random_10   Average Execution Time: 1.513 (sec)  NumChecks: 10
Service: test_random_09   Average Execution Time: 1.780 (sec)  NumChecks: 12
Service: test_random_16   Average Execution Time: 1.811 (sec)  NumChecks: 10
Service: test_random_05   Average Execution Time: 1.913 (sec)  NumChecks: 10
Service: test_random_07   Average Execution Time: 2.767 (sec)  NumChecks: 3
Service: test_random_18   Average Execution Time: 2.972 (sec)  NumChecks: 7
Service: Testping   Average Execution Time: 4.025 (sec)  NumChecks: 1
Service: test_random_06   Average Execution Time: 4.615 (sec)  NumChecks: 12
Service: test_random_01   Average Execution Time: 4.780 (sec)  NumChecks: 9
Service: 2546 forker test   Average Execution Time: 60.015 (sec)  NumChecks: 1

Total Execution Time: 791 (sec)   NumChecks: 4019   Average Time: 0.197 (sec)
```