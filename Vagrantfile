# load .env
Dotenv.load

Vagrant.configure(2) do |config|
  config.vm.box     = 'dummy'
  config.vm.box_url = 'https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box'
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "svt_av1_vm" do |config|
    config.vm.provider :aws do |provider, override|
      provider.region        = ENV['EC2_REGION']
      provider.instance_type = ENV['EC2_INSTANCE_TYPE']
      provider.ami           = ENV['EC2_AMI']
      override.ssh.username  = ENV['EC2_USERNAME']
      # provider.security_groups = "sg-xxxxxxx"

      provider.access_key_id        = ENV['EC2_ACCESS_KEY_ID']
      provider.secret_access_key    = ENV['EC2_SECRET_ACCESS_KEY']
      provider.keypair_name         = ENV['EC2_KEYPAIR']
      override.ssh.private_key_path = ENV['SSH_KEY_PATH']

      override.nfs.functional = false
      override.vm.synced_folder "./sync_folder", "/sync_folder", type: "rsync"      

      #if ARGV[0] == "up" || ARGV[0] == "provision" then
      if ARGV[0] == "up" then
        override.vm.provision :shell do |shell|
          shell.inline = <<-SHELL
            yum -y install \
                centos-release-scl \
                epel-release \
            && yum-config-manager --enable rhel-server-rhscl-7-rpms \
            && yum -y install \
                autoconf \
                automake \
                bzip2 \
                freetype-devel \
                gcc \
                gcc-c++ \
                git \
                libtool \
                make \
                libarchive \
                mercurial \
                pkgconfig \
                zlib-devel \
                unzip \
                wget \
                libcurl \
                openssl \
            && cd /tmp/ \
            && mkdir svt_sources \
            && mkdir svt_build
            # install cmake3.9
            cd /tmp/ \
            && wget https://cmake.org/files/v3.9/cmake-3.9.1.tar.gz \
            && tar xvfz cmake-3.9.1.tar.gz \
            && cd cmake-3.9.1 \
            && ./bootstrap \
            && make -j32 \
            && make install
            # install nasm
            wget -P /tmp/ https://www.nasm.us/pub/nasm/releasebuilds/2.13.03/nasm-2.13.03.tar.bz2 \
            && cd /tmp/svt_sources \
            && tar xvf /tmp/nasm-2.13.03.tar.bz2 \
            && cd nasm-2.13.03 \
            && ./autogen.sh \
            && CC="/usr/bin/gcc" ./configure --prefix="/tmp/svt_build" --bindir="/usr/bin" \
            && make -j32 \
            && make install
            # install yasm
            wget -P /tmp/ http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz \
            && cd /tmp/svt_sources \
            && tar xvf /tmp/yasm-1.3.0.tar.gz \
            && cd yasm-1.3.0 \
            && ./configure --prefix="/tmp/svt_build" --bindir="/usr/bin" \
            && make -j32 \
            && make install
            # install gcc6
            yum install -y devtoolset-6-gcc devtoolset-6-gcc-c++
            # install svt-av1
            cd /tmp/svt_sources \
            && git clone https://github.com/OpenVisualCloud/SVT-AV1.git -b v0.7.5 \
            && cd SVT-AV1/Build/linux \
            && scl enable devtoolset-6 "./build.sh release" \
            && mv /tmp/svt_sources/SVT-AV1/Bin/Release/SvtAv1EncApp /usr/local/bin/ \
            && mv /tmp/svt_sources/SVT-AV1/Bin/Release/SvtAv1DecApp /usr/local/bin/ \
            || exit 1

            cd /tmp/ \
            && mkdir ffmpeg_sources \
            && mkdir ffmpeg_build \
            # install libvpx
            cd /tmp/ffmpeg_sources \
            && git clone https://chromium.googlesource.com/webm/libvpx.git \
            && cd libvpx \
            && git checkout 8ae686757b708cd8df1d10c71586aff5355cfe1e \
            && ./configure --prefix="/tmp/ffmpeg_build" --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm \
            && make -j32 \
            && make install
            # install libaom
            cd /tmp/ffmpeg_sources \
            && git clone https://aomedia.googlesource.com/aom \
            && mkdir aom_build \
            && cd aom_build \
            && PKG_CONFIG_PATH="/tmp/ffmpeg_build/lib/pkgconfig:$PKG_CONFIG_PATH" \
                /usr/local/bin/cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/tmp/ffmpeg_build" -DENABLE_SHARED=off -DENABLE_NASM=on ../aom \
            && make -j32 \
            && make install
            # install libopus
            cd /tmp/ffmpeg_sources \
            && wget -P /tmp/ https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz \
            && tar xzvf /tmp/opus-1.3.1.tar.gz \
            && cd opus-1.3.1 \
            && ./configure --prefix="/tmp/ffmpeg_build" --disable-shared \
            && make -j32 \
            && make install
            # install ffmpeg
            wget -P /tmp/ https://www.ffmpeg.org/releases/ffmpeg-4.1.tar.bz2 \
            && cd /tmp/ffmpeg_sources \
            && tar -jxvf /tmp/ffmpeg-4.1.tar.bz2 \
            && cd ffmpeg-4.1 \
            && PATH="/tmp/bin:$PATH" PKG_CONFIG_PATH="/tmp/ffmpeg_build/lib/pkgconfig:/tmp/ffmpeg_build/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:/usr/local/lib/pkgconfig" \
                ./configure \
                  --prefix="/tmp/ffmpeg_build" \
                  --pkg-config-flags="--static" \
                  --extra-cflags="-I/tmp/ffmpeg_build/include -march=native" \
                  --optflags=-O3 \
                  --enable-static \
                  --disable-shared \
                  --disable-debug \
                  --extra-ldflags="-L/tmp/ffmpeg_build/lib" \
                  --extra-libs=-lpthread \
                  --extra-libs=-lm \
                  --bindir="/usr/bin" \
                  --disable-gpl \
                  --disable-nonfree \
                  --enable-libopus \
                  --disable-libfreetype \
                  --enable-libvpx \
                  --enable-libaom > /sync_folder/ffmpeg_configure.txt \
            && make -j32 \
            && make install
          SHELL
          shell.privileged = true
        end
      end
    end
  end
end


