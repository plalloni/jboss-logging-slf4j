Instalación de LogBack 0.9.26 en JBoss 5.1.0 GA en reemplazo de Log4J
=====================================================================

Convenciones
~~~~~~~~~~~~

  * La ubicación de la instalación de JBoss se referenciará como JBOSS_HOME
  * La ubicación de la instancia de JBoss usada se referenciará como SERVER_HOME (por ejemplo SERVER_HOME=$JBOSS_HOME/server/default)

Instalación
~~~~~~~~~~~

  1. De la distribución de LogBack copiar a $JBOSS_HOME/lib los siguientes archivos:
    - logback-core-0.9.26.jar 
    - logback-classic-0.9.26.jar
    
  2. De la distribución de SLF4J 1.6.1 copiar a $JBOSS_HOME/lib los siguentes archivos:
    - slf4j-api-1.6.1.jar
    - log4j-over-slf4j-1.6.1.jar
    - jcl-over-slf4j-1.6.1.jar
    - jul-to-slf4j-1.6.1.jar
    
  3. Copiar jboss-logging-slf4j-1.jar a $JBOSS_HOME/lib
  
Configuración
~~~~~~~~~~~~~

1. Reemplazar bridge JUL-Log4J por bridge JUL-SLF4J en $SERVER_HOME/conf/bootstrap/logging.xml:

  1. Reemplazar la línea::
    <bean name="LogBridgeHandler" class="org.jboss.logbridge.LogBridgeHandler"/>
    
  2. Por la línea::
    <bean name="LogBridgeHandler" class="org.slf4j.bridge.SLF4JBridgeHandler"/>

2. Configurar LogBack

  Configurar LogBack en archivo $SERVER_HOME/conf/logback.xml.
 
  A modo de ejemplo, se transcribe una configuración de LogBack que simula la configuración de Log4J que trae JBoss mas algunos detalles posibles en LogBack para mostrar excepciones::
    <?xml version="1.0" encoding="UTF-8"?>
    
    <configuration debug="false" scan="true" scanPeriod="30 seconds">
    
       <!-- ================================= -->
       <!-- Preserve messages in a local file -->
       <!-- ================================= -->
    
       <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${jboss.server.log.dir}/server.log</file>
          <append>true</append>
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
             <!-- daily rollover -->
             <fileNamePattern>${jboss.server.log.dir}/server.%d.log</fileNamePattern>
             <!-- keep 30 days' worth of history -->
             <maxHistory>30</maxHistory>
          </rollingPolicy>
          <encoder>
            <pattern>%contextName %date %-5level \(%thread\) [%logger{20}] %message %mdc %xThrowable{full} %n</pattern>
            <!--<pattern>%d %-5p [%c] \(%t\) %m%n</pattern>-->
          </encoder>
       </appender>
    
       <!-- ============================== -->
       <!-- Append messages to the console -->
       <!-- ============================== -->
    
       <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
            <pattern>%contextName %date %-5level \(%thread\) [%logger{20}] %message %mdc %xThrowable{full} %n</pattern>
          </encoder>
       </appender>
    
       <!-- ================ -->
       <!-- Limit categories -->
       <!-- ================ -->
    
       <!-- Limit the org.apache category to INFO as its DEBUG is verbose -->
       <logger name="org.apache" level="INFO"/>
       
       <!-- Limit the jacorb category to WARN as its INFO is verbose -->
       <logger name="jacorb" level="WARN"/>
       
       <!-- Set the logging level of the JSF implementation that uses
          | java.util.logging. The jdk logging levels can be controlled
          | through the org.jboss.logging.log4j.JDKLevel class that
          | in addition to the standard log4j levels it adds support for
          | SEVERE, WARNING, CONFIG, FINE, FINER, FINEST
       -->
       <logger name="javax.enterprise.resource.webcontainer.jsf" level="INFO"/>
       
       <!-- Limit the org.jgroups category to WARN as its INFO is verbose -->
       <logger name="org.jgroups" level="WARN"/>
       
       <!-- Limit the org.quartz category to INFO as its DEBUG is verbose -->
       <logger name="org.quartz" level="INFO"/>
       
       <!-- Limit the com.sun category to INFO as its FINE is verbose -->
       <logger name="com.sun" level="INFO"/>
       
       <!-- Limit the sun category to INFO as its FINE is verbose -->
       <logger name="sun" level="INFO"/>
       
       <!-- Limit the javax.xml.bind category to INFO as its FINE is verbose -->
       <logger name="javax.xml.bind" level="INFO"/>
       
       <!-- Limit JBoss categories - ->
       <logger name="org.jboss" level="INFO"/>
       -->
    
       <!-- Limit the JSR77 categories -->
       <logger name="org.jboss.management" level="INFO"/>
    
       <!-- Limit the verbose facelets compiler -->
       <logger name="facelets.compiler" level="WARN"/>
       
       <!-- Limit the verbose ajax4jsf cache initialization -->
       <logger name="org.ajax4jsf.cache" level="WARN"/>
       
       <!-- Limit the verbose embedded jopr categories -->
       <logger name="org.rhq" level="WARN"/>
       
       <!-- Limit the verbose seam categories -->
       <logger name="org.jboss.seam" level="WARN"/>
       
       <!-- Show the evolution of the DataSource pool in the logs [inUse/Available/Max]
       <logger name="org.jboss.resource.connectionmanager.JBossManagedConnectionPool" level="TRACE"/>
       -->
    
       <!-- Category specifically for Security Audit Provider 
       <logger name="org.jboss.security.audit.providers.LogAuditProvider" additivity="false" level="TRACE">
         <appender-ref ref="AUDIT"/>
       </logger>
       -->
       
       <!-- Limit the org.jboss.serial (jboss-serialization) to INFO as its DEBUG is verbose -->
       <logger name="org.jboss.serial" level="INFO"/>
      
       <!-- Decrease the priority threshold for the org.jboss.varia category
       <logger name="org.jboss.varia" level="DEBUG"/>
       -->
       
       <!-- Enable JBossWS message tracing
       <logger name="org.jboss.ws.core.MessageTrace" level="TRACE"/>
       -->
       
       <!--
          | An example of enabling the custom TRACE level priority that is used
          | by the JBoss internals to diagnose low level details. This example
          | turns on TRACE level msgs for the org.jboss.ejb.plugins package and its
          | subpackages. This will produce A LOT of logging output.
          |
          | Note: since jboss AS 4.2.x, the trace level is supported natively by
          | log4j, so although the custom org.jboss.logging.XLevel priority will
          | still work, there is no need to use it. The two examples that follow
          | will both enable trace logging.
       <logger name="org.jboss.system" level="TRACE"/>
       <logger name="org.jboss.ejb.plugins" level="TRACE"/>
       -->
       
       <!-- ======================= -->
       <!-- Setup the Root category -->
       <!-- ======================= -->
    
          <!-- 
             Set the root logger priority via a system property. Note this is parsed by log4j,
             so the full JBoss system property format is not supported; e.g.
             setting a default via ${jboss.server.log.threshold:WARN} will not work.         
           -->
       <root level="INFO">
          <appender-ref ref="CONSOLE"/>
          <appender-ref ref="FILE"/>
       </root>
    
    </configuration>
  
Ejecución
~~~~~~~~~

En un shell::
    $ cd $JBOSS_HOME
    $ bin/run.sh -b 0.0.0.0 \                     
      -L logback-core-0.9.26.jar \
      -L logback-classic-0.9.26.jar \
      -L jboss-logging-slf4j-1-SNAPSHOT.jar \
      -L slf4j-api-1.6.1.jar \
      -L log4j-over-slf4j-1.6.1.jar \
      -L jcl-over-slf4j-1.6.1.jar \
      -L jul-to-slf4j-1.6.1.jar \
      -Dorg.jboss.logging.Logger.pluginClass=org.lalloni.jboss.logging.slf4j.SLF4JLoggerPlugin \
      -Dlogback.configurationFile="$SERVER_HOME/conf/logback.xml"