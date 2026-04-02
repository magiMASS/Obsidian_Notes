FROM rockylinux:8

RUN dnf -y update && \
    dnf -y install java-11-openjdk-devel unzip dos2unix libnsl && \
    dnf clean all

ENV IDEMIA_HOME=/home/Biosdk/
ENV LD_LIBRARY_PATH=/home/Biosdk/Linux64/

ARG spring_config_label
ARG active_profile
ARG spring_config_url

ENV active_profile_env=${active_profile}
ENV spring_config_label_env=${spring_config_label}
ENV spring_config_url_env=${spring_config_url}

ADD idemia.zip /home/Biosdk/

RUN cd /home/Biosdk/ && unzip idemia.zip && rm idemia.zip

RUN cp /home/Biosdk/conf/* /home/Biosdk/
RUN cp /home/Biosdk/lib/idemiaSdk-1.1.9-SNAPSHOT-jar-with-dependencies.jar /home/Biosdk/
ADD isdk.log /home/Biosdk/
ADD biosdk-services-1.1.3.jar start.sh /home/Biosdk/
RUN chmod +x /home/Biosdk/isdk.log
RUN chmod +x /home/Biosdk/start.sh && dos2unix /home/Biosdk/start.sh

EXPOSE 9099

CMD ["/home/Biosdk/start.sh"]
