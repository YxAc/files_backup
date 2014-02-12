# Advanced Features for Minos

## Build Minos and Install Tank, Supervisor offline

Minos supports building Client and installing Tank, Supervisor offline if your production machines cannot connect the internet.

### Download the required python lib tarballs

Download the following source tarballs to save in ${your_offline_package_directory}

    configobj>=4.7.2 from <https://pypi.python.org/pypi/configobj/>
    django>=1.5.5 from <https://www.djangoproject.com/>
    meld3>=0.6.10 from <https://pypi.python.org/pypi/meld3>
    elementtree>=1.2.6-20050316 from <http://effbot.org/downloads#elementtree>
    pexpect>=3.0 from <https://pypi.python.org/pypi/pexpect/>

### Build Minos offline

    cd minos
    ./build.sh build --offline --package_dir ${your_offline_package_directory}

Then you can refer to `Installing Tank` and `Installing Supervisor` to learn how to install them.

## Using quota_updater in Owl

### Configure quota reportor

    # configure mailsender
    cd minos/owl/utils
    vi mail.py
    edit line 7-9
    e.g.
    self.from_email = 'robot-noreply@example.com'
    self.smtp_host = 'mail.example.com'
    self.password = '123456'

    # configure owl reportor
    cd minos/config/owl
    vi owl_config.py
    edit line 6-11
    e.g.
    QUOTA_REPORT_CLUSTER = ['dptst-example']
    QUOTA_REPORT_ADMINS = 'san.zhang@example.com,'
    QUOTA_ALERT_ADMINS = 'san.zhang@example.com'
    ALLERT_ADMIN_MAIL_ADDR = 'san.zhang@example.com,'

    # configure allert mail lists
    cd minos/config/template
    vi kerberos_ids.txt
    add id-email item
    e.g.
    h_sanzhang san.zhang@example.com

    # configure crontab for quota reportor
    crontab -e
    32 15 * * * ${your_minos_path}/owl/quota_reportor.sh

### Start quota_updater

    cd minos
    ./build.sh start owl --quota_updater

After starting quota_updater, you can view the users' quota information of Hadoop clusters via Owl web interface.

## Using a real HBase cluster in Owl

If you want to use a real hbase cluster, you can build and start owl with `--skip_setup_hbase` to skip setup the default stand-alone hbase. Besides, some other work needs to do for deploying your opentsdb.

### Create hbase table

    cd minos/build/download/opentsdb
    env COMPRESSION=NONE HBASE_HOME=${path_to_hbase_cluster} ./src/create_table.sh

In regard to the path to your real hbase-cluster, you can use the Client component to pack a usable package such as:

    cd minos/client
    ./deploy.sh pack hbase ${hbase_cluster_name}

Then the usable hbase package would be generated at `client/packages/${hbase_cluster_name}`

### Start the opentsdb

    cd minos/build/download/opentsdb
    tsdtmp=${TMPDIR-'/tmp'}/tsd
    mkdir -p "$tsdtmp"
    nohup ./build/tsdb tsd --port=4242 --staticroot=build/staticroot --cachedir="$tsdtmp" --zkquorum="${your_zookeeper_quorum}" --zkbasedir="/hbase/${hbase_cluster_name}" 1>opentsdb.out 2>&1 &

The `zkquorum` is a comma-separated list of hosts serving your zookeeper quorum just like `ip1:port,ip2:port,ip3:port`.

### Start opentsdb collector

    cd minos/config/opentsdb
    vim metrics_collector_config.py
    $opentsdb_extra_args = '--zkquorum=${your_zookeeper_quorum} --zkbasedir=/hbase/${hbase_cluster_name}'
    cd ../../opentsdb
    nohup ./collector.sh &

