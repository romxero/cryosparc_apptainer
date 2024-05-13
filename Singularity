bootstrap: docker
from: nvidia/cuda:11.4.3-devel-ubuntu20.04
stage: build

%labels
        Author William Law
        Version v0.0.2

%files
        entrypoint.bash /entrypoint.bash
        cryosparc.sh    /cryosparc.sh
        cryosparc-server.sh     /app/cryosparc_master/bin/cryosparc-server.sh

%help
    This is an attempt to build CRYOSPARC with MotionCor2 in a singularity/apptainer definition file


%environment
export DEBIAN_FRONTEND=noninteractive
export CRYOSPARC_ROOT_DIR=/app
export CRYOSPARC_VERSION=${CRYOSPARC_VERSION}
export CRYOSPARC_WORKER_DIR=${CRYOSPARC_ROOT_DIR}/cryosparc_worker
export CRYOSPARC_MASTER_DIR=${CRYOSPARC_ROOT_DIR}/cryosparc_master
export CRYOSPARC_VERSION=4.4.1
export MOTIONCOR2_VERSION=${MOTIONCOR2_VERSION}
#export CRYOSPARC_PATCH=231114
export MUNGEUSER=969
export MUNGEGROUP=969
export SLURMUSER=5224
export CRYOSPARC_LICENSE_ID=4df734a2-5300-11ee-9ab8-abff0251d81b




%runscript
   /entrypoint.bash



%test
#need to put tests here
/app/cryosparc_master/bin/cryosparcm test install



%startscript
   /entrypoint.bash



%setup
#    touch /file1
#    touch ${APPTAINER_ROOTFS}/file2


#
#license stuff 
#CRYO_SPARC_LIC=4be03944-949a-11ee-a66b-87114cdf98cb
#export LICENSE_ID=$CRYO_SPARC_LIC

# grab the latest cryosparc tarballs
#curl -L https://get.cryosparc.com/download/master-latest/$LICENSE_ID -o cryosparc_master.tar.gz
#curl -L https://get.cryosparc.com/download/worker-latest/$LICENSE_ID -o cryosparc_worker.tar.gz




%post
export CRYOSPARC_ROOT_DIR=/app
export DEBIAN_FRONTEND=noninteractive
export CRYOSPARC_VERSION=${CRYOSPARC_VERSION}
export CRYOSPARC_WORKER_DIR=${CRYOSPARC_ROOT_DIR}/cryosparc_worker
export CRYOSPARC_MASTER_DIR=${CRYOSPARC_ROOT_DIR}/cryosparc_master
export CRYOSPARC_VERSION=4.4.1
export MOTIONCOR2_VERSION=${MOTIONCOR2_VERSION}
#export CRYOSPARC_PATCH=CRYOSPARC_PATCH
export MOTIONCOR2_VERSION=1.6.4
export MUNGEUSER=969
export MUNGEGROUP=969
export SLURMUSER=5224
export SLURMGROUP=5224
export CRYOSPARC_LICENSE_ID=4df734a2-5300-11ee-9ab8-abff0251d81b

#setup slurm/munge users

groupadd -f -g $SLURMGROUP slurm \
    && useradd  -m -c "SLURM workload manager" -d /var/lib/slurm -u $SLURMUSER -g slurm  -s /bin/bash slurm

groupadd -f -g $MUNGEGROUP munge \
    && useradd -m -c "MUNGE Uid 'N' Gid Emporium" -d /var/lib/munge -u $MUNGEUSER -g munge  -s /sbin/nologin munge \
    && mkdir -p /etc/munge && chown -R munge:$MUNGEGROUP /etc/munge

apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5 2F59B5F99B1BE0B4\
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    apt-utils \
    zip unzip \
    lsof \
    python \
    python3 \
    python3-dev \
    python3-setuptools \
    python3-pip \
    libtiff5 \
    netbase \
    ed \
    curl \
    iputils-ping \
    sudo \
    net-tools \
    openssh-server \
    jq \
    munge \
    ca-certificates \
    gnupg \
  && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg \
  && echo "deb [ signed-by=/usr/share/keyrings/nodesource.gpg ] https://deb.nodesource.com/node_21.x nodistro main" | tee /etc/apt/sources.list.d/node
source.list \
  && apt-get update \
  && apt-get install -y nodejs \
  && curl -fsSL https://repo.mongodb.org/apt/ubuntu/dists/focal/mongodb-org/6.0/Release.gpg | tee /usr/share/keyrings/mongodb.gpg \
  && echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/key-file.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" |
 tee /etc/apt/sources.list.d/mongodb-org-6.0.list \
  && rm -rf /var/lib/apt/lists/* \
  && ln -s /usr/lib/x86_64-linux-gnu/libtiff.so.5 /usr/lib/x86_64-linux-gnu/libtiff.so.3

#symlink slurm-llnl to /etc/slurm to keep enterprise linux folks less confused
ln -s /etc/slurm-llnl /etc/slurm

mkdir -p ${CRYOSPARC_ROOT_DIR}
cd ${CRYOSPARC_ROOT_DIR}


#need ot turn cryosparc license id into a file?!
#RUN --mount=type=secret,id=cryosparc_license_id \
  curl -L https://get.cryosparc.com/download/master-v${CRYOSPARC_VERSION}/${CRYOSPARC_LICENSE_ID} | tar -xz \
        && cd ${CRYOSPARC_MASTER_DIR} \
  && sed -i '/^echo "# Other" >> config.sh$/a echo \"CRYOSPARC_FORCE_USER=true\" >> config.sh' ./install.sh \
  && bash ./install.sh --license "${CRYOSPARC_LICENSE_ID}" --yes --allowroot
  sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_MASTER_DIR}/config.sh


# commenting out the patch stuff for right now

# update patches
#if [ ! -z "${CRYOSPARC_PATCH}" ]; then curl -L https://get.cryosparc.com/patch_get/v${CRYOSPARC_VERSION}+${CRYOSPARC_PATCH}/master -o ${CRYOSPARC_ROOT
#_DIR}/cryosparc_master_patch.tar.gz \
#  && tar -vxzf ${CRYOSPARC_ROOT_DIR}/cryosparc_master_patch.tar.gz --overwrite --strip-components=1 --directory=${CRYOSPARC_MASTER_DIR} \
#  && rm -f ${CRYOSPARC_ROOT_DIR}/cryosparc_master_patch.tar.gz; fi

# patches
#sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_MASTER_DIR}/config.sh
#sed -i 's:    disk_has_space=.*:    disk_has_space="true":g'  ${CRYOSPARC_MASTER_DIR}/bin/cryosparcm

# install worker
cd ${CRYOSPARC_ROOT_DIR}
echo "INSTALLING WORKER"
#--mount=type=secret,id=cryosparc_license_id \
  curl -L https://get.cryosparc.com/download/worker-v${CRYOSPARC_VERSION}/${CRYOSPARC_LICENSE_ID} | tar -xz \
  && cd ${CRYOSPARC_WORKER_DIR} \
  && rm -rf /var/lib/apt/lists/* \
  && ln -s /usr/lib/x86_64-linux-gnu/libtiff.so.5 /usr/lib/x86_64-linux-gnu/libtiff.so.3

#symlink slurm-llnl to /etc/slurm to keep enterprise linux folks less confused
ln -s /etc/slurm-llnl /etc/slurm

mkdir -p ${CRYOSPARC_ROOT_DIR}
cd ${CRYOSPARC_ROOT_DIR}


#need ot turn cryosparc license id into a file?!
#RUN --mount=type=secret,id=cryosparc_license_id \
  curl -L https://get.cryosparc.com/download/master-v${CRYOSPARC_VERSION}/${CRYOSPARC_LICENSE_ID} | tar -xz \
        && cd ${CRYOSPARC_MASTER_DIR} \
  && sed -i '/^echo "# Other" >> config.sh$/a echo \"CRYOSPARC_FORCE_USER=true\" >> config.sh' ./install.sh \
  && bash ./install.sh --license "${CRYOSPARC_LICENSE_ID}" --yes --allowroot
  sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_MASTER_DIR}/config.sh


# update patches
if [ ! -z "${CRYOSPARC_PATCH}" ]; then curl -L https://get.cryosparc.com/patch_get/v${CRYOSPARC_VERSION}+${CRYOSPARC_PATCH}/master -o ${CRYOSPARC_ROOT
_DIR}/cryosparc_master_patch.tar.gz \
  && tar -vxzf ${CRYOSPARC_ROOT_DIR}/cryosparc_master_patch.tar.gz --overwrite --strip-components=1 --directory=${CRYOSPARC_MASTER_DIR} \
  && rm -f ${CRYOSPARC_ROOT_DIR}/cryosparc_master_patch.tar.gz; fi

# patches
sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_MASTER_DIR}/config.sh
sed -i 's:    disk_has_space=.*:    disk_has_space="true":g'  ${CRYOSPARC_MASTER_DIR}/bin/cryosparcm

# install worker
cd ${CRYOSPARC_ROOT_DIR}
echo "INSTALLING WORKER"
#--mount=type=secret,id=cryosparc_license_id \
  curl -L https://get.cryosparc.com/download/worker-v${CRYOSPARC_VERSION}/${CRYOSPARC_LICENSE_ID} | tar -xz \
  && cd ${CRYOSPARC_WORKER_DIR} \
  && bash ./install.sh --license ${CRYOSPARC_LICENSE_ID} --yes
  sed -i '/^echo "# Other" >> config.sh$/a echo \"CRYOSPARC_FORCE_USER=true\" >> config.sh' ./install.sh
  sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_WORKER_DIR}/config.sh

# update patches
if [ ! -z "${CRYOSPARC_PATCH}" ]; then curl -L https://get.cryosparc.com/patch_get/v${CRYOSPARC_VERSION}+${CRYOSPARC_PATCH}/worker -o ${CRYOSPARC_ROOT
_DIR}/cryosparc_worker_patch.tar.gz \
  && tar -vxzf ${CRYOSPARC_ROOT_DIR}/cryosparc_worker_patch.tar.gz --overwrite --strip-components=1 --directory=${CRYOSPARC_WORKER_DIR} \
  && rm -f ${CRYOSPARC_ROOT_DIR}/cryosparc_worker_patch.tar.gz; fi
  sed -i 's/^export CRYOSPARC_LICENSE_ID=.*$/export CRYOSPARC_LICENSE_ID=TBD/g' ${CRYOSPARC_WORKER_DIR}/config.sh

####
## install motioncor
####


cd /usr/local/bin \
  && curl -L 'https://drive.google.com/uc?export=download&id=1hskY_AbXVgrl_BUIjWokDNLZK0c1FLxF' > MotionCor2_${MOTIONCOR2_VERSION}.zip \
  && unzip MotionCor2_${MOTIONCOR2_VERSION}.zip \
  && rm -f MotionCor2_${MOTIONCOR2_VERSION}.zip \
  && ln -sf MotionCor2_${MOTIONCOR2_VERSION}-Cuda100 MotionCor2




%arguments

#CRYOSPARC_PATCH=231114
MOTIONCOR2_VERSION=1.6.4
CRYOSPARC_VERSION=4.4.1
MUNGEUSER=969
MUNGEGROUP=969
SLURMUSER=5224
SLURMGROUP=5224
#CRYOSPARC_LICENSE_ID=4df734a2-5300-11ee-9ab8-abff0251d81b

