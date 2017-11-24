Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu-16.04-amd64'

  config.vm.provider :virtualbox do |vb|
    vb.linked_clone = true
    vb.memory = 256
    vb.customize ['modifyvm', :id, '--cableconnected1', 'on']
  end

  config.vm.define :gateway do |config|
    config.vm.hostname = 'gateway'
    config.vm.network :private_network, ip: '10.10.10.2'
    config.vm.provision :shell, path: 'gateway.sh'
  end

  config.vm.define :ubuntu do |config|
    config.vm.box = 'empty'
    config.vm.network :private_network, mac: '080027000001', ip: '10.10.10.0', auto_config: false
    config.vm.provider :virtualbox do |vb, config|
      # let vagrant known that the guest does not have the guest additions nor a functional vboxsf or shared folders.
      vb.check_guest_additions = false
      vb.functional_vboxsf = false
      config.vm.synced_folder '.', '/vagrant', disabled: true

      # configure for PXE boot.
      vb.customize ['modifyvm', :id, '--boot1', 'net']
      vb.customize ['modifyvm', :id, '--boot2', 'disk']
      vb.customize ['modifyvm', :id, '--biospxedebug', 'on']
      vb.customize ['modifyvm', :id, '--cableconnected2', 'on']
      vb.customize ['modifyvm', :id, '--nicbootprio2', '1']
      vb.customize ['modifyvm', :id, "--nictype2", '82540EM'] # Must be an Intel card (as-of VB 5.1 we cannot Intel PXE boot from a virtio-net card).
      # NB the valid empty System UUID is 00000000-0000-4000-8000-000000000000
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemUuid', '00000000-0000-4000-8000-000000000001']
    end
  end

  config.vm.define :tcl do |config|
    config.vm.box = 'empty'
    config.vm.network :private_network, mac: '080027000002', ip: '10.10.10.0', auto_config: false
    config.ssh.insert_key = false
    config.ssh.shell = '/bin/sh' # TCL uses BusyBox ash instead of bash.
    config.vm.provider :virtualbox do |vb, config|
      # let vagrant known that the guest does not have the guest additions nor a functional vboxsf or shared folders.
      vb.check_guest_additions = false
      vb.functional_vboxsf = false
      config.vm.synced_folder '.', '/vagrant', disabled: true
      
      # configure for PXE boot.
      vb.customize ['modifyvm', :id, '--boot1', 'net']
      vb.customize ['modifyvm', :id, '--boot2', 'disk']
      vb.customize ['modifyvm', :id, '--biospxedebug', 'on']
      vb.customize ['modifyvm', :id, '--cableconnected2', 'on']
      vb.customize ['modifyvm', :id, '--nicbootprio2', '1']
      vb.customize ['modifyvm', :id, "--nictype2", '82540EM'] # Must be an Intel card (as-of VB 5.1 we cannot Intel PXE boot from a virtio-net card).

      # set some BIOS settings that will help us identify this particular machine.
      #
      #   VirtualBox          | Linux
      #   --------------------+----------------------------------------------
      #   DmiSystemVendor     | /sys/devices/virtual/dmi/id/sys_vendor
      #   DmiSystemProduct    | /sys/devices/virtual/dmi/id/product_name
      #   DmiSystemVersion    | /sys/devices/virtual/dmi/id/product_version
      #   DmiSystemSerial     | /sys/devices/virtual/dmi/id/product_serial
      #   DmiSystemSKU        | TODO where do we read this from?
      #   DmiSystemUuid       | /sys/devices/virtual/dmi/id/product_uuid
      #   DmiChassisVendor    | /sys/devices/virtual/dmi/id/chassis_asset_tag
      #   DmiChassisType      | /sys/devices/virtual/dmi/id/chassis_type
      #   DmiChassisVersion   ! /sys/devices/virtual/dmi/id/chassis_version
      #   DmiChassisSerial    ! /sys/devices/virtual/dmi/id/chassis_serial
      #   DmiChassisAssetTag  | /sys/devices/virtual/dmi/id/chassis_asset_tag
      #
      # See https://www.virtualbox.org/svn/vbox/trunk/src/VBox/Devices/PC/DevPcBios.cpp
      # See https://www.virtualbox.org/browser/vbox/trunk/src/VBox/Devices/PC/BIOS
      # See https://www.virtualbox.org/svn/vbox/trunk/src/VBox/Devices/PC/BIOS/bios.c
      #
      # NB the VirtualBox BIOS is based on Plex86/Boch/QEMU.
      # NB dump extradata with VBoxManage getextradata $(cat .vagrant/machines/client/virtualbox/id)
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemVendor',    'your vendor name here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemProduct',   'your product name here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemVersion',   'your product version here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemSerial',    'your product serial number here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemSKU',       'your product SKU here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiSystemUuid',      '00000000-0000-4000-8000-000000000002']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiChassisVendor',   'your chassis vendor name here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiChassisType',     '1'] 
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiChassisVersion',  'your chassis version here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiChassisSerial',   'your chassis serial number here']
      vb.customize ['setextradata', :id, 'VBoxInternal/Devices/pcbios/0/Config/DmiChassisAssetTag', 'your chassis asset tag here']
    end

    # make sure we use the gateway machine as this machine default gateway.
    # NB the 10.0.2/24 network is the default VirtualBox NAT network, we remove its default route. 
    #config.vm.provision :shell, run: 'always', inline: 'ip route del $(ip route list 0/0 | grep 10\\.0\\.2\\.)'
    config.vm.provision :shell, run: 'always', inline: "set -eux; i=$(route -n | grep -E '^0\\.0\\.0\\.0\\s+10\\.0\\.2\\.' | sed -E 's,.+(eth.+),\\1,'); route del default $i"

    # dump some useful information.
    config.vm.provision :shell, inline: '''
        set -x
        uname -a
        route -n
        #ip route
        #ip route get 8.8.8.8
        cat /proc/cmdline
        cat /sys/devices/virtual/dmi/id/sys_vendor
        cat /sys/devices/virtual/dmi/id/product_name
        cat /sys/devices/virtual/dmi/id/product_version
        cat /sys/devices/virtual/dmi/id/product_serial
        cat /sys/devices/virtual/dmi/id/product_uuid
        cat /sys/devices/virtual/dmi/id/chassis_vendor
        cat /sys/devices/virtual/dmi/id/chassis_type
        cat /sys/devices/virtual/dmi/id/chassis_version
        cat /sys/devices/virtual/dmi/id/chassis_serial
        cat /sys/devices/virtual/dmi/id/chassis_asset_tag
      '''
  end
end