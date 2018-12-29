FROM centos:6.8
LABEL version=18.12.28
LABEL jdkVersion=1.6.0.45
LABEL weblogicVersion=10.3.6
LABEL patch="only yum update,not weblogic "

#############install jdk1.6.45
#copy jdk
WORKDIR /usr

COPY jdk-6u45-linux-x64.bin .

#yum update and install jdk
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo \
    && yum update -y \ 
    && rm -rf /var/cache/yum \
    && chmod +x jdk-6u45-linux-x64.bin \
    && sh jdk-6u45-linux-x64.bin \
    && rm -rf jdk-6u45-linux-x64.bin \
    && chmod -R 777 /usr/jdk1.6.0_45






###############install weblogic

ENV WLS_PKG wls1036_generic.jar
ENV CONFIG_JVM_ARGS -Djava.security.egd=file:/dev/./urandom

# weblogic user
# ------------------------------------------------------------
RUN mkdir /u01 && \
    chmod a+xr /u01 && \
    useradd -b /u01 -m -s /bin/bash weblogic 

# Copy packages
COPY $WLS_PKG /u01/
COPY wls-silent.xml /u01/


# Adjust file permissions, go to /u01 as user 'weblogic' to proceed with WLS installation
RUN chown weblogic:weblogic -R /u01
WORKDIR /u01
USER weblogic

#set env
ENV JAVA_HOME=/usr/jdk1.6.0_45 \
    CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar \
	PATH=${PATH}:$JAVA_HOME


# Installation of WebLogic 
RUN $JAVA_HOME/bin/java -jar $WLS_PKG -mode=silent -silent_xml=/u01/wls-silent.xml && \ 
	rm $WLS_PKG /u01/wls-silent.xml 

WORKDIR /u01/oracle/


###################create domain
# WLS Configuration
# -------------------------------

#设定weblogic用户名和密码，这里等级比python脚本等级高
ENV ADMIN_USER weblogic
ENV ADMIN_PASSWORD Cloud#.2019
ENV ADMIN_PORT 7001
ENV NM_PORT 5556
ENV USER_MEM_ARGS -Xms256m -Xmx512m -XX:MaxPermSize=1024m
ENV EXTRA_JAVA_PROPERTIES $EXTRA_JAVA_PROPERTIES -Djava.security.egd=file:///dev/urandom
ENV PATH $PATH:/usr/jdk1.6.0_45/bin:/u01/oracle/weblogic/oracle_common/common/bin

# 复制文件
COPY create-wls-domain.py /u01/oracle/


# Root commands
USER root
RUN echo ". /u01/oracle/weblogic/user_projects/domains/base_domain/bin/setDomainEnv.sh" >> /root/.bashrc && \
    echo "export PATH=$PATH:/u01/oracle/weblogic/wlserver_10.3/common/bin:/u01/oracle/weblogic/user_projects/domains/base_domain/bin" >> /root/.bashrc


# Configuration of WLS Domain
USER weblogic
WORKDIR /u01/oracle/weblogic

#先删除/u01/oracle/weblogic/wlserver_10.3/common/bin/commEnv.sh脚本中对java_home的重置
RUN sed -i 's/JAVA_HOME=""//g'  /u01/oracle/weblogic/wlserver_10.3/common/bin/commEnv.sh && \
    /u01/oracle/weblogic/wlserver_10.3/common/bin/wlst.sh -skipWLSModuleScanning /u01/oracle/create-wls-domain.py && \
    mkdir -p /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security && \
    echo "username=${ADMIN_USER}" > /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security/boot.properties && \ 
    echo "password=${ADMIN_PASSWORD}" >> /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security/boot.properties && \
    echo ". /u01/oracle/weblogic/user_projects/domains/base_domain/bin/setDomainEnv.sh" >> /u01/oracle/.bashrc && \ 
    echo "export PATH=$PATH:/u01/oracle/weblogic/wlserver_10.3/common/bin:/u01/oracle/weblogic/user_projects/domains/base_domain/bin" >> /u01/oracle/.bashrc 

# Expose Node Manager default port, and also default http/https ports for admin console
EXPOSE $NM_PORT $ADMIN_PORT 


# Final setup
WORKDIR /u01/oracle

ENV PATH $PATH:/u01/oracle/weblogic/wlserver_10.3/common/bin:/u01/oracle/weblogic/user_projects/domains/base_domain/bin:/u01/oracle

# Define default command to start bash. 
CMD ["startWebLogic.sh"]

