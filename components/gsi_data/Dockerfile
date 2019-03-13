FROM centos:latest
MAINTAINER Michael Kavulich <kavulich@ucar.edu>
#
RUN yum -y update
RUN yum -y install wget
RUN mkdir /gsi_data
#
# Get GSI tutorial data
#
#RUN mkdir -p /gsi_data/ \
# && wget  https://dtcenter.org/container_nwp_tutorial/tar_files/gsi_obs.tar -P /gsi_data/ \
# && tar -xf /gsi_data/gsi_obs.tar -C /gsi_data/
#
# Download CRTM files
RUN cd /gsi_data/ \
 && curl -L -O https://dtcenter.org/com-GSI/users/downloads/GSI_releases/CRTM_v2.3.0.tar.gz \
 && tar -xf CRTM_v2.3.0.tar.gz \
 && rm CRTM_v2.3.0.tar.gz
VOLUME /gsi_data
CMD ["true"]
