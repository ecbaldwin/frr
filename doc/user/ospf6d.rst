.. _ospfv3:

******
OSPFv3
******

*ospf6d* is a daemon support OSPF version 3 for IPv6 network. OSPF for IPv6 is
described in :rfc:`2740`.

.. _ospf6-router:

OSPF6 router
============

.. clicmd:: router ospf6 [vrf NAME]

.. clicmd:: ospf6 router-id A.B.C.D

   Set router's Router-ID.

.. clicmd:: timers throttle spf (0-600000) (0-600000) (0-600000)

   This command sets the initial `delay`, the `initial-holdtime`
   and the `maximum-holdtime` between when SPF is calculated and the
   event which triggered the calculation. The times are specified in
   milliseconds and must be in the range of 0 to 600000 milliseconds.

   The `delay` specifies the minimum amount of time to delay SPF
   calculation (hence it affects how long SPF calculation is delayed after
   an event which occurs outside of the holdtime of any previous SPF
   calculation, and also serves as a minimum holdtime).

   Consecutive SPF calculations will always be separated by at least
   'hold-time' milliseconds. The hold-time is adaptive and initially is
   set to the `initial-holdtime` configured with the above command.
   Events which occur within the holdtime of the previous SPF calculation
   will cause the holdtime to be increased by `initial-holdtime`, bounded
   by the `maximum-holdtime` configured with this command. If the adaptive
   hold-time elapses without any SPF-triggering event occurring then
   the current holdtime is reset to the `initial-holdtime`.

   .. code-block:: frr

      router ospf6
       timers throttle spf 200 400 10000


   In this example, the `delay` is set to 200ms, the initial holdtime is set
   to 400ms and the `maximum holdtime` to 10s. Hence there will always be at
   least 200ms between an event which requires SPF calculation and the actual
   SPF calculation. Further consecutive SPF calculations will always be
   separated by between 400ms to 10s, the hold-time increasing by 400ms each
   time an SPF-triggering event occurs within the hold-time of the previous
   SPF calculation.

.. clicmd:: auto-cost reference-bandwidth COST


   This sets the reference bandwidth for cost calculations, where this
   bandwidth is considered equivalent to an OSPF cost of 1, specified in
   Mbits/s. The default is 100Mbit/s (i.e. a link of bandwidth 100Mbit/s
   or higher will have a cost of 1. Cost of lower bandwidth links will be
   scaled with reference to this cost).

   This configuration setting MUST be consistent across all routers
   within the OSPF domain.

.. clicmd:: maximum-paths (1-64)

   Use this command to control the maximum number of parallel routes that
   OSPFv3 can support. The default is 64.

.. clicmd:: write-multiplier (1-100)

   Use this command to tune the amount of work done in the packet read and
   write threads before relinquishing control. The parameter is the number
   of packets to process before returning. The default value of this parameter
   is 20.

.. clicmd:: clear ipv6 ospf6 process [vrf NAME]

   This command clears up the database and routing tables and resets the
   neighborship by restarting the interface state machine. This will be
   helpful when there is a change in router-id and if user wants the router-id
   change to take effect, user can use this cli instead of restarting the
   ospf6d daemon.

.. clicmd:: clear ipv6 ospf6 [vrf NAME] interface [IFNAME]

   This command restarts the interface state machine for all interfaces in the
   VRF or only for the specific interface if ``IFNAME`` is specified.

ASBR Summarisation Support in OSPFv3
====================================

   External routes in OSPFv3 are carried by type 5/7 LSA (external LSAs).
   External LSAs are generated by ASBR (Autonomous System Boundary Router).
   Large topology database requires a large amount of router memory, which
   slows down all processes, including SPF calculations.
   It is necessary to reduce the size of the OSPFv3 topology database,
   especially in a large network. Summarising routes keeps the routing
   tables smaller and easier to troubleshoot.

   External route summarization must be configured on ASBR.
   Stub area do not allow ASBR because they don’t allow type 5 LSAs.

   An ASBR will inject a summary route into the OSPFv3 domain.

   Summary route will only be advertised if you have at least one subnet
   that falls within the summary range.

   Users will be allowed an option in the CLI to not advertise range of
   ipv6 prefixes as well.

   The configuration of ASBR Summarisation is supported using the CLI command

.. clicmd:: summary-address X:X::X:X/M [tag (1-4294967295)] [{metric (0-16777215) | metric-type (1-2)}]

   This command will advertise a single External LSA on behalf of all the
   prefixes falling under this range configured by the CLI.
   The user is allowed to configure tag, metric and metric-type as well.
   By default, tag is not configured, default metric as 20 and metric-type
   as type-2 gets advertised.
   A summary route is created when one or more specific routes are learned and
   removed when no more specific route exist.
   The summary route is also installed in the local system with Null0 as
   next-hop to avoid leaking traffic.

.. clicmd:: no summary-address X:X::X:X/M [tag (1-4294967295)] [{metric (0-16777215) | metric-type (1-2)}]

   This command can be used to remove the summarisation configuration.
   This will flush the single External LSA if it was originated and advertise
   the External LSAs for all the existing individual prefixes.

.. clicmd:: summary-address X:X::X:X/M no-advertise

   This command can be used when user do not want to advertise a certain
   range of prefixes using the no-advertise option.
   This command when configured will flush all the existing external LSAs
   falling under this range.

.. clicmd:: no summary-address X:X::X:X/M no-advertise

   This command can be used to remove the previous configuration.
   When configured, tt will resume originating external LSAs for all the prefixes
   falling under the configured range.

.. clicmd:: aggregation timer (5-1800)

   The summarisation command takes effect after the aggregation timer expires.
   By default the value of this timer is 5 seconds. User can modify the time
   after which the external LSAs should get originated using this command.

.. clicmd:: no aggregation timer (5-1800)

   This command removes the timer configuration. It reverts back to default
   5 second timer.

.. clicmd:: show ipv6 ospf6 summary-address [detail] [json]

   This command can be used to see all the summary-address related information.
   When detail option is used, it shows all the prefixes falling under each
   summary-configuration apart from other information.

.. _ospf6-area:

OSPF6 area
==========

.. clicmd:: area A.B.C.D range X:X::X:X/M [<advertise|not-advertise|cost (0-16777215)>]

.. clicmd:: area (0-4294967295) range X:X::X:X/M [<advertise|not-advertise|cost (0-16777215)>]

    Summarize a group of internal subnets into a single Inter-Area-Prefix LSA.
    This command can only be used at the area boundary (ABR router).

    By default, the metric of the summary route is calculated as the highest
    metric among the summarized routes. The `cost` option, however, can be used
    to set an explicit metric.

    The `not-advertise` option, when present, prevents the summary route from
    being advertised, effectively filtering the summarized routes.

.. clicmd:: area A.B.C.D nssa [no-summary]

.. clicmd:: area (0-4294967295) nssa [no-summary] [default-information-originate [metric-type (1-2)] [metric (0-16777214)]]

   Configure the area to be a NSSA (Not-So-Stubby Area).

   The following functionalities are implemented as per RFC 3101:

   1. Advertising Type-7 LSA into NSSA area when external route is
      redistributed into OSPFv3.
   2. Processing Type-7 LSA received from neighbor and installing route in the
      route table.
   3. Support for NSSA ABR functionality which is generating Type-5 LSA when
      backbone area is configured. Currently translation of Type-7 LSA to
      Type-5 LSA is enabled by default.
   4. Support for NSSA Translator functionality when there are multiple NSSA
      ABR in an area.

   An NSSA ABR can be configured with the `no-summary` option to prevent the
   advertisement of summaries into the area. In that case, a single Type-3 LSA
   containing a default route is originated into the NSSA.

   NSSA ABRs and ASBRs can be configured with `default-information-originate`
   option to originate a Type-7 default route into the NSSA area. In the case
   of NSSA ASBRs, the origination of the default route is conditioned to the
   existence of a default route in the RIB that wasn't learned via the OSPF
   protocol.

.. clicmd:: area A.B.C.D export-list NAME

.. clicmd:: area (0-4294967295) export-list NAME

   Filter Type-3 summary-LSAs announced to other areas originated from intra-
   area paths from specified area.

   .. code-block:: frr

      router ospf6
       area 0.0.0.10 export-list foo
      !
      ipv6 access-list foo permit 2001:db8:1000::/64
      ipv6 access-list foo deny any

   With example above any intra-area paths from area 0.0.0.10 and from range
   2001:db8::/32 (for example 2001:db8:1::/64 and 2001:db8:2::/64) are announced
   into other areas as Type-3 summary-LSA's, but any others (for example
   2001:200::/48) aren't.

   This command is only relevant if the router is an ABR for the specified
   area.

.. clicmd:: area A.B.C.D import-list NAME

.. clicmd:: area (0-4294967295) import-list NAME

   Same as export-list, but it applies to paths announced into specified area
   as Type-3 summary-LSAs.

.. clicmd:: area A.B.C.D filter-list prefix NAME in

.. clicmd:: area A.B.C.D filter-list prefix NAME out

.. clicmd:: area (0-4294967295) filter-list prefix NAME in

.. clicmd:: area (0-4294967295) filter-list prefix NAME out

   Filtering Type-3 summary-LSAs to/from area using prefix lists. This command
   makes sense in ABR only.

.. _ospf6-interface:

OSPF6 interface
===============

.. clicmd:: ipv6 ospf6 area <A.B.C.D|(0-4294967295)>

   Enable OSPFv3 on the interface and add it to the specified area.

.. clicmd:: ipv6 ospf6 cost COST

   Sets interface's output cost. Default value depends on the interface
   bandwidth and on the auto-cost reference bandwidth.

.. clicmd:: ipv6 ospf6 hello-interval HELLOINTERVAL

   Sets interface's Hello Interval. Default 10

.. clicmd:: ipv6 ospf6 dead-interval DEADINTERVAL

   Sets interface's Router Dead Interval. Default value is 40.

.. clicmd:: ipv6 ospf6 retransmit-interval RETRANSMITINTERVAL

   Sets interface's Rxmt Interval. Default value is 5.

.. clicmd:: ipv6 ospf6 priority PRIORITY

   Sets interface's Router Priority. Default value is 1.

.. clicmd:: ipv6 ospf6 transmit-delay TRANSMITDELAY

   Sets interface's Inf-Trans-Delay. Default value is 1.

.. clicmd:: ipv6 ospf6 network (broadcast|point-to-point)

   Set explicitly network type for specified interface.

OSPF6 route-map
===============

Usage of *ospfd6*'s route-map support.

.. clicmd:: set metric [+|-](0-4294967295)

   Set a metric for matched route when sending announcement. Use plus (+) sign
   to add a metric value to an existing metric. Use minus (-) sign to
   substract a metric value from an existing metric.

.. _redistribute-routes-to-ospf6:

Redistribute routes to OSPF6
============================

.. clicmd:: redistribute <babel|bgp|connected|isis|kernel|openfabric|ripng|sharp|static|table> [metric-type (1-2)] [metric (0-16777214)] [route-map WORD]

   Redistribute routes of the specified protocol or kind into OSPFv3, with the
   metric type and metric set if specified, filtering the routes using the
   given route-map if specified.

.. clicmd:: default-information originate [{always|metric (0-16777214)|metric-type (1-2)|route-map WORD}]

   The command injects default route in the connected areas. The always
   argument injects the default route regardless of it being present in the
   router. Metric values and route-map can also be specified optionally.

Graceful Restart
================

.. clicmd:: graceful-restart [grace-period (1-1800)]


   Configure Graceful Restart (RFC 5187) restarting support.
   When enabled, the default grace period is 120 seconds.

   To perform a graceful shutdown, the "graceful-restart prepare ipv6 ospf"
   EXEC-level command needs to be issued before restarting the ospf6d daemon.

.. clicmd:: graceful-restart helper enable [A.B.C.D]


   Configure Graceful Restart (RFC 5187) helper support.
   By default, helper support is disabled for all neighbours.
   This config enables/disables helper support on this router
   for all neighbours.
   To enable/disable helper support for a specific
   neighbour, the router-id (A.B.C.D) has to be specified.

.. clicmd:: graceful-restart helper strict-lsa-checking


   If 'strict-lsa-checking' is configured then the helper will
   abort the Graceful Restart when a LSA change occurs which
   affects the restarting router.
   By default 'strict-lsa-checking' is enabled"

.. clicmd:: graceful-restart helper supported-grace-time (10-1800)


   Supports as HELPER for configured grace period.

.. clicmd:: graceful-restart helper planned-only


   It helps to support as HELPER only for planned
   restarts. By default, it supports both planned and
   unplanned outages.

.. clicmd:: graceful-restart prepare ipv6 ospf


   Initiate a graceful restart for all OSPFv3 instances configured with the
   "graceful-restart" command. The ospf6d daemon should be restarted during
   the instance-specific grace period, otherwise the graceful restart will fail.

   This is an EXEC-level command.


.. _showing-ospf6-information:

Showing OSPF6 information
=========================

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] [json]

   Show information on a variety of general OSPFv3 and area state and
   configuration information. JSON output can be obtained by appending 'json'
   to the end of command.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] database [<detail|dump|internal>] [json]

   This command shows LSAs present in the LSDB. There are three view options.
   These options helps in viewing all the parameters of the LSAs. JSON output
   can be obtained by appending 'json' to the end of command. JSON option is
   not applicable with 'dump' option.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] database <router|network|inter-prefix|inter-router|as-external|group-membership|type-7|link|intra-prefix> [json]

   These options filters out the LSA based on its type. The three views options
   works here as well. JSON output can be obtained by appending 'json' to the
   end of command.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] database adv-router A.B.C.D linkstate-id A.B.C.D [json]

   The LSAs additinally can also be filtered with the linkstate-id and
   advertising-router fields. We can use the LSA type filter and views with
   this command as well and visa-versa. JSON output can be obtained by
   appending 'json' to the end of command.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] database self-originated [json]

   This command is used to filter the LSAs which are originated by the present
   router. All the other filters are applicable here as well.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] interface [json]

   To see OSPF interface configuration like costs. JSON output can be
   obtained by appending "json" in the end.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] neighbor [json]

   Shows state and chosen (Backup) DR of neighbor. JSON output can be
   obtained by appending 'json' at the end.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] interface traffic [json]

   Shows counts of different packets that have been recieved and transmitted
   by the interfaces. JSON output can be obtained by appending "json" at the
   end.

.. clicmd:: show ipv6 route ospf6

   This command shows internal routing table.

.. clicmd:: show ipv6 ospf6 zebra [json]

   Shows state about what is being redistributed between zebra and OSPF6.
   JSON output can be obtained by appending "json" at the end.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] redistribute [json]

   Shows the routes which are redistributed by the router. JSON output can
   be obtained by appending 'json' at the end.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] route [<intra-area|inter-area|external-1|external-2|X:X::X:X|X:X::X:X/M|detail|summary>] [json]

   This command displays the ospfv3 routing table as determined by the most
   recent SPF calculations. Options are provided to view the different types
   of routes. Other than the standard view there are two other options, detail
   and summary. JSON output can be obtained by appending 'json' to the end of
   command.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] route X:X::X:X/M match [detail] [json]

   The additional match option will match the given address to the destination
   of the routes, and return the result accordingly.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] interface [IFNAME] prefix [detail|<X:X::X:X|X:X::X:X/M> [<match|detail>]] [json]

   This command shows the prefixes present in the interface routing table.
   Interface name can also be given. JSON output can be obtained by appending
   'json' to the end of command.

.. clicmd:: show ipv6 ospf6 [vrf <NAME|all>] spf tree [json]

   This commands shows the spf tree from the recent spf calculation with the
   calling router as the root. If json is appended in the end, we can get the
   tree in JSON format. Each area that the router belongs to has it's own
   JSON object, with each router having "cost", "isLeafNode" and "children" as
   arguments.

.. clicmd:: show ipv6 ospf6 graceful-restart helper [detail] [json]

   This command shows the graceful-restart helper details including helper
   configuration parameters.

.. _ospf6-debugging:

OSPFv3 Debugging
================

The following debug commands are supported:

.. clicmd:: debug ospf6 abr

   Toggle OSPFv3 ABR debugging messages.

.. clicmd:: debug ospf6 asbr

   Toggle OSPFv3 ASBR debugging messages.

.. clicmd:: debug ospf6 border-routers {router-id [A.B.C.D] | area-id [A.B.C.D]}

   Toggle OSPFv3 border router debugging messages. This can be specified for a
   router with specific Router-ID/Area-ID.

.. clicmd:: debug ospf6 flooding

   Toggle OSPFv3 flooding debugging messages.

.. clicmd:: debug ospf6 interface

   Toggle OSPFv3 interface related debugging messages.

.. clicmd:: debug ospf6 lsa

   Toggle OSPFv3 Link State Advertisements debugging messages.

.. clicmd:: debug ospf6 lsa aggregation

   Toggle OSPFv3 Link State Advertisements summarization debugging messages.

.. clicmd:: debug ospf6 message

   Toggle OSPFv3 message exchange debugging messages.

.. clicmd:: debug ospf6 neighbor

   Toggle OSPFv3 neighbor interaction debugging messages.

.. clicmd:: debug ospf6 nssa

   Toggle OSPFv3 Not So Stubby Area (NSSA) debugging messages.

.. clicmd:: debug ospf6 route

   Toggle OSPFv3 routes debugging messages.

.. clicmd:: debug ospf6 spf

   Toggle OSPFv3 Shortest Path calculation debugging messages.

.. clicmd:: debug ospf6 zebra

   Toggle OSPFv3 zebra interaction debugging messages.

.. clicmd:: debug ospf6 graceful-restart

   Toggle OSPFv3 graceful-restart helper debugging messages.

Sample configuration
====================

Example of ospf6d configured on one interface and area:

.. code-block:: frr

   interface eth0
    ipv6 ospf6 area 0.0.0.0
    ipv6 ospf6 instance-id 0
   !
   router ospf6
    ospf6 router-id 212.17.55.53
    area 0.0.0.0 range 2001:770:105:2::/64
   !


Larger example with policy and various options set:


.. code-block:: frr

   debug ospf6 neighbor state
   !
   interface fxp0
    ipv6 ospf6 area 0.0.0.0
    ipv6 ospf6 cost 1
    ipv6 ospf6 hello-interval 10
    ipv6 ospf6 dead-interval 40
    ipv6 ospf6 retransmit-interval 5
    ipv6 ospf6 priority 0
    ipv6 ospf6 transmit-delay 1
    ipv6 ospf6 instance-id 0
   !
   interface lo0
    ipv6 ospf6 cost 1
    ipv6 ospf6 hello-interval 10
    ipv6 ospf6 dead-interval 40
    ipv6 ospf6 retransmit-interval 5
    ipv6 ospf6 priority 1
    ipv6 ospf6 transmit-delay 1
    ipv6 ospf6 instance-id 0
   !
   router ospf6
    router-id 255.1.1.1
    redistribute static route-map static-ospf6
   !
   access-list access4 permit 127.0.0.1/32
   !
   ipv6 access-list access6 permit 3ffe:501::/32
   ipv6 access-list access6 permit 2001:200::/48
   ipv6 access-list access6 permit ::1/128
   !
   ipv6 prefix-list test-prefix seq 1000 deny any
   !
   route-map static-ospf6 permit 10
    match ipv6 address prefix-list test-prefix
    set metric-type type-2
    set metric 2000
   !
   line vty
    access-class access4
    ipv6 access-class access6
    exec-timeout 0 0
   !
