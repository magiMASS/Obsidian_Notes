FROM rockylinux:8

RUN dnf -y update && \
    dnf -y install libnsl && \
    dnf clean all

ENV IDEMIA_HOME=/home/Biosdk/
ENV LD_LIBRARY_PATH=/home/Biosdk/Linux64/

# ✅ Create base directory
RUN mkdir -p /home/Biosdk/

# ✅ Copy License and config separately
COPY License/ /home/Biosdk/License/
COPY slm_conf/ /home/Biosdk/slm_conf/

# ✅ Install RPM
RUN dnf install -y /home/Biosdk/License/multiprotect-5.1.2-1.x86_64.rpm

# ✅ Move config
RUN mv /home/Biosdk/slm_conf /usr/sbin/Multiprotect/

# ✅ OpenShift compatibility (CRITICAL)
RUN chmod -R 777 /home/Biosdk /usr/sbin/Multiprotect

# ✅ Run daemon as main process
CMD ["/usr/sbin/Multiprotect/Multiprotect_License_Daemon"]
