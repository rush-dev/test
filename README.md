shcluster
masterapps
deployment-apps

| rest /services/deployment/server/clients
| foreach applications.*.restartSplunkd [eval Apps=if(isnotnull('<<FIELD>>'), mvappend(Apps, "<<MATCHSTR>>"), Apps)]
| foreach serverClasses.*.restartSplunkd [eval ServerClasses=if(isnotnull('<<FIELD>>'), mvappend(ServerClasses, "<<MATCHSTR>>"), ServerClasses)]
| eval client = lower(dns)
| rex field=utsname "(?<os>[^\-]+)\-(?<arch>.+)"
| eval os = case(os == "linux", "Linux", os == "windows", "Windows", arch == "sun4u", "Solaris", arch == "sun4v", "Solaris")
| rename dns AS client, averagePhoneHomeInterval AS PhoneHomeInterval
| stats values(Apps) AS Apps, values(ServerClasses) AS ServerClasses count by client ip os arch splunkVersion build clientName splunk_server PhoneHomeInterval lastPhoneHomeTime
| fieldformat lastPhoneHomeTime=strftime(lastPhoneHomeTime, "%F %T %Z")
| eval missing=(now() - lastPhoneHomeTime - PhoneHomeInterval)
| eval missing=if(missing < 0, 0, missing)
| eval Missing=case (missing==0, "No", missing==1, "Yes")
| fields - missing
