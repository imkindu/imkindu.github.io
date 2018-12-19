








wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el6.x86_64.rpm
  106  ll
  107  yum install erlang-19.0.4-1.el6.x86_64.rpm 
  108  erl -v
  109  cd /usr/local/
  110  ll
  111  rpm -ql erlang
  112  rpm -ql erlang | head
  113  which erlang
  114  which erl
  115  pwd
  116  cd tools/
  117  ll
  118  wget https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v3_6_15/rabbitmq-server-3.6.15-1.el6.noarch.rpm
  119  ll
  120  rpm -qa socat
  121  yum install socat
  122  yum install epel-release
  123  yum install socat
  124  ll
  125  yum install rabbitmq-server-3.6.15-1.el6.noarch.rpm 
  126  rpm -ql rabbitmq-server
  127  service rabbitmq-server start







[root@centos100 tools]# rpm -ql rabbitmq-server
/etc/logrotate.d/rabbitmq-server
/etc/rabbitmq
/etc/rc.d/init.d/rabbitmq-server
/usr/lib/ocf/resource.d/rabbitmq/rabbitmq-server
/usr/lib/ocf/resource.d/rabbitmq/rabbitmq-server-ha
/usr/lib/rabbitmq/bin/rabbitmq-defaults
/usr/lib/rabbitmq/bin/rabbitmq-env
/usr/lib/rabbitmq/bin/rabbitmq-plugins
/usr/lib/rabbitmq/bin/rabbitmq-server
/usr/lib/rabbitmq/bin/rabbitmqctl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/background_gc.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/delegate.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/delegate_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/dtree.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/file_handle_cache.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/file_handle_cache_stats.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/gatherer.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/gm.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/lqueue.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/mirrored_supervisor_sups.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/mnesia_sync.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/pg_local.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit.app
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_access_control.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_alarm.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_amqqueue_process.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_amqqueue_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_amqqueue_sup_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_auth_mechanism_amqplain.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_auth_mechanism_cr_demo.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_auth_mechanism_plain.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_autoheal.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_binding.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_boot_steps.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_channel_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_channel_sup_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_cli.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_client_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_connection_helper_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_connection_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_control_main.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_control_pbe.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_ctl_usage.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_dead_letter.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_diagnostics.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_direct.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_disk_monitor.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_epmd_monitor.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_error_logger.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_error_logger_file_h.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_parameters.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_type_direct.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_type_fanout.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_type_headers.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_type_invalid.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_exchange_type_topic.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_file.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_framing.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_guid.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_hipe.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_limiter.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_log.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_memory_monitor.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_coordinator.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_master.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_misc.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_mode.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_mode_all.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_mode_exactly.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_mode_nodes.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_slave.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mirror_queue_sync.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mnesia.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_mnesia_rename.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_msg_file.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_msg_store.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_msg_store_ets_index.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_msg_store_gc.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_node_monitor.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_parameter_validation.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_password.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_password_hashing_md5.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_password_hashing_sha256.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_password_hashing_sha512.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_pbe.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_plugins.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_plugins_main.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_plugins_usage.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_policies.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_policy.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_prelaunch.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_prequeue.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_priority_queue.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_consumers.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_index.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_location_client_local.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_location_min_masters.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_location_random.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_location_validator.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_queue_master_location_misc.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_recovery_terms.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_registry.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_resource_monitor_misc.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_restartable_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_router.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_runtime_parameters.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_sasl_report_file_h.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_ssl.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_table.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_trace.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_upgrade.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_upgrade_functions.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_variable_queue.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_version.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_vhost.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/rabbit_vm.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/supervised_lifecycle.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/tcp_listener.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/tcp_listener_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/truncate.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/vm_memory_monitor.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/worker_pool.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/worker_pool_sup.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/ebin/worker_pool_worker.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/gm_specs.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/old_builtin_types.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/rabbit.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/rabbit_cli.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/rabbit_framing.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/rabbit_misc.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/include/rabbit_msg_store.hrl
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/README
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/amqp_client-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/cowboy-1.0.3.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/cowlib-1.0.1.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/mochiweb-2.13.1.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbit_common-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_amqp1_0-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_auth_backend_ldap-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_auth_mechanism_ssl-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_consistent_hash_exchange-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_event_exchange-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_federation-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_federation_management-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_jms_topic_exchange-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_management-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_management_agent-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_management_visualiser-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_mqtt-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_recent_history_exchange-1.2.1.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_sharding-0.1.0.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_shovel-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_shovel_management-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_stomp-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_top-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_tracing-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_trust_store-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_web_dispatch-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_web_stomp-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/rabbitmq_web_stomp_examples-3.6.6.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/ranch-1.2.1.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/sockjs-0.3.4.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/plugins/webmachine-1.10.3.ez
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/sbin/rabbitmq-defaults
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/sbin/rabbitmq-env
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/sbin/rabbitmq-plugins
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/sbin/rabbitmq-server
/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.6/sbin/rabbitmqctl
/usr/sbin/rabbitmq-plugins
/usr/sbin/rabbitmq-server
/usr/sbin/rabbitmqctl
/usr/share/doc/rabbitmq-server-3.6.6
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-APACHE2-ExplorerCanvas
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-APL2-Rebar
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-APL2-Stomp-Websocket
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-BSD-base64js
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-BSD-glMatrix
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-EPL-OTP
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-EJS10
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-Erlware-Commons
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-Flot
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-Mochi
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-Mochiweb
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-Sammy060
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-SockJS
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MIT-jQuery164
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MPL-RabbitMQ
/usr/share/doc/rabbitmq-server-3.6.6/LICENSE-MPL2
/usr/share/doc/rabbitmq-server-3.6.6/README
/usr/share/doc/rabbitmq-server-3.6.6/rabbitmq.config.example
/usr/share/doc/rabbitmq-server-3.6.6/set_rabbitmq_policy.sh.example
/usr/share/man/man1/rabbitmq-plugins.1.gz
/usr/share/man/man1/rabbitmq-server.1.gz
/usr/share/man/man1/rabbitmqctl.1.gz
/usr/share/man/man5/rabbitmq-env.conf.5.gz
/var/lib/rabbitmq
/var/lib/rabbitmq/mnesia
/var/log/rabbitmq
