## Tomcat Session Clustering

* 지금 설명하는 Tomcat Session Clustering은 외장 Tomcat에 관한 설정이다.  <br>

Tomcat 설치 후 conf 폴더 하위의 server.xml을 수정한다.  
Cluster 단어를 search하여 해당 부분에 하기와 같이 작성한다.  

한 서버에 톰캣이 여러대 있는 경우, receiver port는 겹치지 않도록 한다.  
Manager는 session cluster를 all-to-all로 지원하는 Delta Manager를 사용하였다.

~~~bash
        <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                 channelSendOptions="6">
          <!--
          <Manager className="org.apache.catalina.ha.session.BackupManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"
                   mapSendOptions="6"/>
          -->
          <Manager className="org.apache.catalina.ha.session.DeltaManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"/>
          
          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                        address="228.0.0.4"
                        port="45564"
                        frequency="500"
                        dropTime="3000"/>
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                      address="auto"
                      port="4000"
                      selectorTimeout="100"
                      maxThreads="6"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.ThroughputInterceptor"/>
          </Channel>

          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=".*\.gif|.*\.js|.*\.jpeg|.*\.jpg|.*\.png|.*\.htm|.*\.html|.*\.css|.*\.txt"/>

          <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                    tempDir="/tmp/war-temp/"
                    deployDir="/tmp/war-deploy/"
                    watchDir="/tmp/war-listen/"
                    watchEnabled="false"/>

          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
        </Cluster>

~~~

<br>
위의 설정을 추가하였으면, 본인 프로젝트의 "프로젝트/src/main/webapp/WEB-INF/" 경로에 web.xml 추가  
해당 web.xml에 하기 태그 추가 (필수!)

```xml
 <distributable/>
```


* 참고 : http://tomcat.apache.org/tomcat-8.5-doc/cluster-howto.html
