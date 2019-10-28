FROM continuumio/miniconda
RUN apt-get update -y && apt-get -y install gcc git libgl1-mesa-glx
COPY ./environment.yml /tmp/environment.yml

RUN conda env create -f /tmp/environment.yml -n cea
RUN echo "source activate cea" > ~/.bashrc
RUN /bin/bash -c "source activate cea && \
    pip install git+https://github.com/architecture-building-systems/CityEnergyAnalyst.git@docker-test && \
    cea extract-reference-case --destination ~/ && \
    cea-config demand --scenario ~/reference-case-open/baseline && \
    cea-config radiation --daysim-bin-directory /root/Daysim/bin"

# bugfix for matplotlib, see here: https://stackoverflow.com/questions/37604289/tkinter-tclerror-no-display-name-and-no-display-environment-variable
RUN mkdir -p ~/.config/matplotlib && echo "backend: Agg" > ~/.config/matplotlib/matplotlibrc

# build the DAYSIM stuff
# first: build radiance
RUN apt-get -y install cmake build-essential libgl1-mesa-dev freeglut3-dev
RUN git clone https://github.com/MITSustainableDesignLab/Daysim.git
RUN mkdir build && cd build && cmake -DBUILD_HEADLESS=on -DCMAKE_INSTALL_PREFIX=$HOME/Daysim ../Daysim && make && make install

# next: build the Daysim specific stuff
COPY ./CMakeLists.txt /Daysim/CMakeLists.txt
RUN cd build && cmake -DBUILD_HEADLESS=on -DCMAKE_INSTALL_PREFIX=$HOME/Daysim ../Daysim && make && mv ./bin/rtrace ./bin/rtrace_dc && cp ./bin/* /root/Daysim/bin

# use the environment - no need to source because it's the only environment anyway...
ENV PATH "/opt/conda/envs/cea/bin:/root/Daysim/bin:$PATH"