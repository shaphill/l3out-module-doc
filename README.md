# Terraform ACI Provider L3Out Module Usage Guide
## General
### What is a module?
Modules provide a way of packaging Terraform configuration to be reused or shared. Modules will typically contain a main ```.tf``` file containing all the resources that have been packaged as well as a ```variables.tf``` file to detail the necessary variables for running. 

### How do I use a module?
To instantiate a module, all you need to do is define a module block:
````
module "name" {
  source      = "path/to/module"
  tenant_dn   = aci_tenant.tenant.id
  name        = "name"
  description = "Created by l3out module"
  vrf_dn      = aci_vrf.vrf.id
  
  parameter   = value
}
````

The module block needs a source field to determine where to fetch the module from--this can be a local or universal path like Github. To start using the module, you will need to add parameters into the module block, which are essentially variables for the resources within the module to use. Parameters can be various types: numbers, booleans, strings, lists, maps, sets, or objects. These are outlined within the module's [```variables.tf```](https://github.com/shrsr/terraform-aci-modules/blob/l3out/l3out/variables.tf) file as well as [down below](#inputs):

## Using the L3Out module
This module contains two structures, complete and simplified.
 - Complete: 
	 - The L3Out resources can be created using a comprehensive approach which gives the user the complete set of options available in the provider today.
	 - Uses the [logical\_nodes\_profile](#input\_logical\_node\_profiles) input
	 - [```l3out.tf```](https://github.com/shrsr/terraform-aci-modules/blob/l3out/l3out/l3out.tf) contains complete module configuration
- Simplified: 
	- Logical interface profiles can be created using a simplified approach with minimal syntax
	- Splits the [logical\_nodes\_profile](#input\_logical\_node\_profiles) input block into [vpc](#input\_vpcs), [floating_svi](#input\_floating\_svi), [ospf_interface_profile](#input\_ospf\_interface\_profile), [bgp_peers](#input\_bgp\_peers), and [nodes](#input\_nodes) input variables
	- [```simplified.tf```](https://github.com/shrsr/terraform-aci-modules/blob/l3out/l3out/simplified.tf) contains the simplified module configuration


We have a comprehensive list of [example use cases](#examples) below detailing both simplified and complete syntax where applicable. The [Complete](#complete-syntax) and [Simplified](#simplified-syntax) syntax examples include large configuration files demonstrating how to use the entire module. There is also a [walkthrough](#guided-example) for writing configuration for the module to create an L3Out EPG using OSPF with vPCs and SVIs. 
## Examples

### Complete Syntax
The complete syntax offers the full capability of the Terraform ACI provider, which also means it can be quite complicated. This version will use the full [logical\_node\_profiles](#input\_logical\_node\_profiles) variable to define most of the L3Out policies. A detailed example configuration is available [here](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/complete_syntax/main.tf).  

### Simplified Syntax
The simplified syntax offers a more minimal approach to defining logical interface profiles by separating the [logical\_nodes\_profile](#input\_logical\_node\_profiles) variable into specific blocks. We've included various examples using the simplified syntax below:
 - [Floating SVI with BGP and OSPF](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/simplified/floating_svi_ospf_bgp.tf)
 - [SVI with BGP and OSPF](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/simplified/svi_ospf_bgp.tf)
 - [vPC, Floating SVI and Nodes with BGP enabled](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/simplified/main.tf)
 - [Nodes with OSPF](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/simplified/nodes_ospf.tf)
 - [vPC with OSPF](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/comprehensive/simplified/vpc_ospf.tf)

### Floating SVI
Floating SVI IPv4/v6 addresses can be defined in tandem which will automatically create two floating SVI logical interface profiles for each address family. Multiple anchor nodes can be defined with IPv4/v6 addresses in tandem and will automatically create the appropriate IP version floating SVI interface profile.
```
  floating_svi = {
  domain_dn        = aci_vmm_domain.virtual_domain.id
  floating_ip      = "19.1.2.1/24"
  floating_ipv6    = "2001:db1:a::15/64"
  vlan             = "4"
  forged_transmit  = false
  mac_change       = false
  promiscuous_mode = true
  anchor_nodes = [
    {
      pod_id     = "1"
      node_id    = "110"
      ip_address = "19.1.1.18/24"
      vlan       = "1"
    },
    {
      pod_id       = "1"
      node_id      = "114"
      ip_address   = "19.1.1.21/24"
      ipv6_address = "2001:db1:a::18/64"
      vlan         = "1"
    }
  ]
}
```
Examples using both complete and simplified syntax are below:

 - [BGP](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/floating_svi/physical.tf)
 - [OSPF](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/floating_svi/virtual.tf)

### OSPF
Examples of both complete and simplified syntax are [here](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/ospf/main.tf)


### Port Channel SVI
Either a Routed Sub-Interface, Routed Interface, or SVI for a particular node can be defined minimally using `port` or `channel` along with its `ip` address family. To create a routed sub-interface, we assign a `vlan`. To create a routed interface we don't assign a `vlan`. To create a SVI we assign a `vlan` and set `svi = true`. IPv4/v6 addresses can be defined in tandem for an interface type, automatically creating two logical interface profiles for the said interface type for each address family. 
```
nodes = [
  {
    node_id          = "101"
    pod_id           = "1"
    router_id        = "102.102.102.102"
    loopback_address = "172.16.31.101"
    interfaces = [
      {
        port = "1/13"
        ip   = "14.1.1.2/24"
        ipv6 = "2001:db8:b::2/64"
      },
      {
        port = "1/12"
        ip   = "10.1.1.49/24"
        ipv6 = "2001:db8:c::2/64"
        vlan = "2"
      },
      {
        channel                = "channel-one"
        ip                     = "14.14.14.1/24"
        secondary_ip_addresses = ["14.15.14.1/24", "14.16.14.1/24", "14.17.14.1/24"]
        svi                    = true
        vlan = "3"
      }
    ]
  }
]
```
An example is below:
 - [Port Channel SVI](https://github.com/shrsr/terraform-aci-modules/blob/l3out/examples/l3out/port_channel_svi/main.tf)


### Shared L3Out
An example including both tenant configurations is available [here](https://github.com/shrsr/terraform-aci-modules/tree/l3out/examples/l3out/shared_l3out/main.tf)


### vPC
vPCs can be defined with both IPv4/v6 addresses for either side--this will automatically create logical interface profiles for each address family by associating themselves to the nodes defined in the vPC block. If you are using vPCs, use this block instead of the SVI block as it will assume SVI usage. The `channel` parameter is used to form the `target_dn` for the [aci_l3out_path_attachment](https://registry.terraform.io/providers/CiscoDevNet/aci/latest/docs/resources/l3out_path_attachment) resource as follows: `"topology/pod-{pod_id}/protpaths-{node_id}/pathep-[{channel}]"`.
```
vpcs = [
  {
    pod_id = 1
    nodes = [
      {
        node_id            = "121"
        router_id          = "1.1.1.101"
        router_id_loopback = "no"
        loopback_address   = "172.16.32.101"
      }
    ]
    interfaces = [
      {
        channel = "channel_vpc1"
        mtu     = "9000"
        vlan    = "1"
        side_a = {
          ip                       = "19.1.2.18/24"
          ipv6                     = "2000:db2:a::15/64"
          secondary_ip_addresses   = ["19.1.2.17/24"]
          secondary_ipv6_addresses = ["2000:db2:a::17/64"]
        }
        side_b = {
          ip                       = "19.1.2.19/24"
          ipv6                     = "2000:db2:a::16/64"
          secondary_ip_addresses   = ["19.1.2.21/24"]
          secondary_ipv6_addresses = ["2000:db2:a::18/64"]
        }
      }
    ]
  }
]
```
An example is [here](https://github.com/shrsr/terraform-aci-modules/tree/l3out/examples/l3out/vpc)

## Guided Example
### Configuration
![Topology](https://i.imgur.com/Qoh3jgj.jpeg)
In this demo, we are deploying an L3Out EPG to allow traffic originating from the "Host" to travel through the ACI fabric and reach the "Endpoint". We will also configure OSPF routing on leaves 103, 104 as shown below:
```
OSPF Area 0

Node Profiles
 - node-103 (103.103.103.103)
 - node-104 (104.104.104.104)

Interface Profile (SVI)
 - Side A (10.0.1.2/24)
 - Side B (10.0.1.3/24)

L3Out EPG External Subnets
 - 0.0.0.0/0
 - 10.0.1.0/24
```
#### Prerequisites
In order to create this L3Out, you must have the appropriate policy objects in place. In this document we will not cover the creation of VRF, BD, Application Profiles, or Access Policies (L3 Domain etc).  For resources on how to configure these objects, view documentation [here](https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/6x/l3-configuration/cisco-apic-layer-3-networking-configuration-guide-60x.html). 
#### Module Configuration
To start using the module, we define the module block and provide the source, tenant, L3Out name, VRF, and the L3 domain. 
```
module  "l3out_ospf" {
  source        =  "./modules/l3out"
  tenant_dn     =  aci_tenant.tenant.id
  name          =  "l3-ospf"
  vrf_dn        =  aci_vrf.vrf.id
  l3_domain_dn  =  data.aci_l3_domain_profile.l3-domain.id
```
To begin the OSPF configuration, we create the [ospf](#input\_ospf) block and pass in our parameters.
```
  ospf  =  {
    area_id    =  "0"
    area_type  =  "regular"
  }
```
Next, because we are configuring vPCs, we will use the [vPCs](#input\_vpcs) variable block to define all of our interface and node profiles, including our SVI configuration. We leave the `ospf_interface_profile` blank here to pass in an empty interface policy. If you want to use a specific OSPF interface policy, you must create the [aci_ospf_interface_policy](https://registry.terraform.io/providers/CiscoDevNet/aci/latest/docs/resources/ospf_interface_policy) resource outside of the module and add it in the `ospf_interface_profile` block as `ospf_interface_policy = aci_ospf_interface_policy.{name}.id`
```
  vpcs  =  [
    {
      ospf_interface_profile  = {
      }
      pod_id  =  1
      nodes  = [
        {
          node_id             =  "103"
          router_id           =  "103.103.103.103"
          router_id_loopback  =  "yes"
        },
        {
          node_id             =  "104"
          router_id           =  "104.104.104.104"
          router_id_loopback  =  "yes"
        }
      ]
      interfaces  = [
        {
          channel  =  "sp-vpc"
          vlan     =  "107"
          mtu      =  "1500"
          side_a  = {
            ip  =  "10.0.1.2/24"
          }
          side_b  = {
            ip  =  "10.0.1.3/24"
          }
        }
      ]
    }
  ]
```
Lastly, we specify the L3 EPG that will be receiving traffic from our host EPG. We also define external subnets and a consumed contract to permit traffic. 
```
  external_epgs  =  [
    {
      name = "l3-ospf-epg"
      consumed_contracts = [
        aci_contract.contract.id
      ]
      subnets  = [
        {
          ip     =  "0.0.0.0/0"
          scope  = ["import-security"]
        },
        {
          ip     =  "10.0.1.0/24"
          scope  = ["import-security"]
        }
      ]
    }
  ]
```
With all the configuration done, 
## Inputs
### Required
| Name | Type | Default |
|------|------|---------|
| <span id="input_name">[name](#input\_name)</span>  <br> L3Out Name | `string` | n/a | yes |
| <span id="input_tenant_dn">[tenant\_dn](#input\_tenant\_dn)</span> <br> Distinguished name of the parent Tenant object | `string` | n/a | yes |

### Optional
| Name | Type | Default |
|------|------|---------|
| <span id="input_alias">[alias](#input\_alias)</span> <br> Name alias of the L3Out object | `string` | `""` |
| <span id="input_annotation">[annotation](#input\_annotation)</span>  <br> Annotation of the L3Out object | `string` | `"orchestrator:terraform"` |
| <span id="input_bgp">[bgp](#input\_bgp)</span>  <br> L3Out BGP protocol | `bool` | `false` |
| <span id="input_bgp_peers">[bgp\_peers](#input\_bgp\_peers)</span>  <br> BGP Peers definition at the global level. It gets pushed to all the nodes in the module when `loopback_as_source` is `true`. If `loopback_as_source` is `false` it gets pushed to all the interfaces. | <pre>list(object(<br>    {<br>      loopback_as_source  = optional(bool)<br>      ip_address          = optional(string)<br>      ipv6_address        = optional(string)<br>      address_control     = optional(list(string))<br>      allowed_self_as_cnt = optional(string)<br>      annotation          = optional(string)<br>      bgp_controls = optional(object(<br>        {<br>          allow_self_as     = optional(bool)<br>          as_override       = optional(bool)<br>          dis_peer_as_check = optional(bool)<br>          nh_self           = optional(bool)<br>          send_com          = optional(bool)<br>          send_ext_com      = optional(bool)<br>        }<br>      ))<br>      alias                  = optional(string)<br>      password               = optional(string)<br>      peer_controls          = optional(list(string))<br>      private_as_control     = optional(list(string))<br>      ebgp_multihop_ttl      = optional(string)<br>      weight                 = optional(string)<br>      as_number              = optional(string)<br>      local_asn              = optional(string)<br>      local_as_number_config = optional(string)<br>      admin_state            = optional(string)<br>      route_control_profiles = optional(list(object({<br>        direction = string<br>        target_dn = string<br>        }<br>      )))<br>    }<br>  ))</pre> | <pre>[<br>  {<br>    "loopback_as_source": true<br>  }<br>]</pre> |
| <span id="input_consumer_label">[consumer\_label](#input\_consumer\_label)</span>  <br> L3Out Consumer Label | `string` | `""` |
| <span id="input_default_route_leak_policy">[default\_route\_leak\_policy](#input\_default\_route\_leak\_policy)</span>  <br> Route Profile for Interleak Policy | <pre>object({<br>    criteria = optional(string)<br>    always   = optional(string)<br>    scope    = optional(list(string))<br>    }<br>  )</pre> | `null` | no |
| <span id="input_description"></span> [description](#input\_description) <br> L3Out description | `string` | `""` | no |
| <span id="input_external_epgs"></span> [external\_epgs](#input\_external\_epgs) <br> L3Out External EPGs Block | <pre>list(object(<br>    {<br>      annotation                   = optional(string)<br>      description                  = optional(string)<br>      exception_tag                = optional(string)<br>      label_match_criteria         = optional(string)<br>      alias                        = optional(string)<br>      name                         = string<br>      preferred_group_member       = optional(bool)<br>      qos_class                    = optional(string)<br>      target_dscp                  = optional(string)<br>      provided_contracts           = optional(list(string))<br>      consumed_contract_interfaces = optional(list(string))<br>      consumed_contracts           = optional(list(string))<br>      taboo_contracts              = optional(list(string))<br>      inherited_contracts          = optional(list(string))<br>      contract_masters = optional(list(object(<br>        {<br>          external_epg = string<br>          l3out        = string<br>        }<br>      )))<br>      route_control_profiles = optional(list(object(<br>        {<br>          direction    = string<br>          route_map_dn = string<br>        }<br>      )))<br>      subnets = optional(list(object(<br>        {<br>          ip        = string<br>          aggregate = optional(string)<br>          alias     = optional(string)<br>          scope     = list(string)<br>          route_control_profiles = optional(list(object(<br>            {<br>              direction    = string<br>              route_map_dn = string<br>            }<br>          )))<br>        }<br>      )))<br>    }<br>  ))</pre> | `[]` | no |
| <span id="input_fallback_route_group_dns"></span> [fallback\_route\_group\_dns](#input\_fallback\_route\_group\_dns) | `list(string)` | `[]` | no |
| <span id="input_floating_svi"></span> [floating\_svi](#input\_floating\_svi) <br> Simplified Block for defining L3out Floating SVI | <pre>object(<br>    {<br>      domain_dn                         = optional(string)<br>      floating_ip                       = optional(string)<br>      floating_ipv6                     = optional(string)<br>      forged_transmit                   = optional(bool)<br>      mac_change                        = optional(bool)<br>      promiscuous_mode                  = optional(bool)<br>      floating_secondary_ip_addresses   = optional(list(string))<br>      floating_secondary_ipv6_addresses = optional(list(string))<br>      vlan                              = optional(string)<br>      ospf_interface_profile = optional(object(<br>        {<br>          authentication_key    = optional(string)<br>          authentication_key_id = optional(string)<br>          authentication_type   = optional(string)<br>          ospf_interface_policy = optional(string)<br>          description           = optional(string)<br>          annotation            = optional(string)<br>        }<br>      ))<br>      anchor_nodes = optional(list(object(<br>        {<br>          pod_id                   = string<br>          node_id                  = string<br>          ip_address               = optional(string)<br>          ipv6_address             = optional(string)<br>          secondary_ip_addresses   = optional(list(string))<br>          secondary_ipv6_addresses = optional(list(string))<br>          description              = optional(string)<br>          mtu                      = optional(string)<br>          vlan                     = optional(string)<br>          encap_scope              = optional(string)<br>          mode                     = optional(string)<br>          annotation               = optional(string)<br>          autostate                = optional(string)<br>          ipv6_dad                 = optional(string)<br>          link_local_address       = optional(string)<br>          mac                      = optional(string)<br>          target_dscp              = optional(string)<br>          bgp_peers = optional(list(object(<br>            {<br>              ip_address          = optional(string)<br>              ipv6_address        = optional(string)<br>              address_control     = optional(list(string))<br>              allowed_self_as_cnt = optional(string)<br>              annotation          = optional(string)<br>              bgp_controls = optional(object(<br>                {<br>                  allow_self_as     = optional(bool)<br>                  as_override       = optional(bool)<br>                  dis_peer_as_check = optional(bool)<br>                  nh_self           = optional(bool)<br>                  send_com          = optional(bool)<br>                  send_ext_com      = optional(bool)<br>                }<br>              ))<br>              alias                  = optional(string)<br>              password               = optional(string)<br>              peer_controls          = optional(list(string))<br>              private_as_control     = optional(list(string))<br>              ebgp_multihop_ttl      = optional(string)<br>              weight                 = optional(string)<br>              as_number              = optional(string)<br>              local_asn              = optional(string)<br>              local_as_number_config = optional(string)<br>              admin_state            = optional(string)<br>              route_control_profiles = optional(list(object({<br>                direction = string<br>                target_dn = string<br>                }<br>              )))<br>            }<br>          )))<br>        }<br>      )))<br>  })</pre> | <pre>{<br>  "anchor_nodes": []<br>}</pre> | no |
| <span id="input_import_route_control"></span> [import\_route\_control](#input\_import\_route\_control) <br> Import route control profile | `bool` | `false` | no |
| <span id="input_l3_domain_dn"></span> [l3\_domain\_dn](#input\_l3\_domain\_dn) <br> Distinguished name of the L3 Domain | `string` | `""` | no |
| <span id="input_logical_node_profiles"></span> [logical\_node\_profiles](#input\_logical\_node\_profiles) <br> Logical Node Profiles Block | <pre>list(object(<br>    {<br>      annotation  = optional(string)<br>      description = optional(string)<br>      alias       = optional(string)<br>      name        = string<br>      tag         = optional(string)<br>      target_dscp = optional(string)<br>      bgp_peers_nodes = optional(list(object({<br>        ip_address          = string<br>        address_control     = optional(list(string))<br>        allowed_self_as_cnt = optional(string)<br>        annotation          = optional(string)<br>        bgp_controls = optional(object(<br>          {<br>            allow_self_as     = optional(bool)<br>            as_override       = optional(bool)<br>            dis_peer_as_check = optional(bool)<br>            nh_self           = optional(bool)<br>            send_com          = optional(bool)<br>            send_ext_com      = optional(bool)<br>        }))<br>        alias                  = optional(string)<br>        password               = optional(string)<br>        peer_controls          = optional(list(string))<br>        private_as_control     = optional(list(string))<br>        ebgp_multihop_ttl      = optional(string)<br>        weight                 = optional(string)<br>        as_number              = optional(string)<br>        local_asn              = optional(string)<br>        local_as_number_config = optional(string)<br>        admin_state            = optional(string)<br>        route_control_profiles = optional(list(object({<br>          direction = string<br>          target_dn = string<br>          }<br>        )))<br>        }<br>      )))<br>      bgp_protocol_profile = optional(object(<br>        {<br>          bgp_timers     = optional(string)<br>          as_path_policy = optional(string)<br>        }<br>      ))<br>      bfd_multihop_protocol_profile = optional(object(<br>        {<br>          authentication_type           = optional(string)<br>          authentication_key_id         = optional(string)<br>          authentication_key            = optional(string)<br>          bfd_multihop_node_policy_name = string<br>        }<br>      ))<br>      nodes = optional(list(object(<br>        {<br>          node_id            = string<br>          pod_id             = string<br>          router_id          = optional(string)<br>          router_id_loopback = optional(string)<br>          loopback_address   = optional(string)<br>          static_routes = optional(list(object({<br>            ip                  = string<br>            alias               = optional(string)<br>            description         = optional(string)<br>            fallback_preference = optional(string)<br>            route_control       = optional(bool)<br>            track_policy        = optional(string)<br>            next_hop_addresses = optional(list(object({<br>              next_hop_ip          = string<br>              annotation           = optional(string)<br>              alias                = optional(string)<br>              preference           = optional(string)<br>              nexthop_profile_type = optional(string)<br>              description          = optional(string)<br>              track_member         = optional(string)<br>              track_policy         = optional(string)<br>              }<br>            )))<br>            }<br>          )))<br>        }<br>      )))<br>      interfaces = optional(list(object(<br>        {<br>          name = string<br>          ospf_interface_profile = optional(object(<br>            {<br>              authentication_key    = optional(string)<br>              authentication_key_id = optional(string)<br>              authentication_type   = optional(string)<br>              ospf_interface_policy = optional(string)<br>              description           = optional(string)<br>              annotation            = optional(string)<br>            }<br>          ))<br>          bfd_interface_profile = optional(object(<br>            {<br>              authentication_key     = optional(string)<br>              authentication_key_id  = optional(string)<br>              interface_profile_type = optional(string)<br>              description            = optional(string)<br>              annotation             = optional(string)<br>              bfd_interface_policy   = optional(string)<br>            }<br>          ))<br>          bfd_multihop_interface_profile = optional(object(<br>            {<br>              authentication_key                 = optional(string)<br>              authentication_key_id              = optional(string)<br>              authentication_type                = optional(string)<br>              bfd_multihop_interface_policy_name = string<br>            }<br>          ))<br>          hsrp = optional(object(<br>            {<br>              annotation = optional(string)<br>              alias      = optional(string)<br>              version    = optional(string)<br>              hsrp_groups = optional(list(object({<br>                name                  = string<br>                annotation            = optional(string)<br>                description           = optional(string)<br>                address_family        = optional(string)<br>                group_id              = optional(string)<br>                ip                    = optional(string)<br>                ip_obtain_mode        = optional(string)<br>                mac                   = optional(string)<br>                alias                 = optional(string)<br>                secondary_virtual_ips = optional(list(string))<br>                }<br>              )))<br>            }<br>          ))<br>          netflow_monitor_policies = optional(list(object(<br>            {<br>              filter_type                 = string<br>              netflow_monitor_policy_name = string<br>            }<br>          )))<br>          egress_data_policy_dn  = optional(string)<br>          ingress_data_policy_dn = optional(string)<br>          custom_qos_policy_dn   = optional(string)<br>          nd_policy_dn           = optional(string)<br>          paths = optional(list(object(<br>            {<br>              interface_type     = string<br>              path_type          = string<br>              pod_id             = string<br>              node_id            = string<br>              node2_id           = optional(string)<br>              interface_id       = string<br>              ip_address         = optional(string)<br>              mtu                = optional(string)<br>              encap              = optional(string)<br>              encap_scope        = optional(string)<br>              mode               = optional(string)<br>              annotation         = optional(string)<br>              autostate          = optional(string)<br>              ipv6_dad           = optional(string)<br>              link_local_address = optional(string)<br>              mac                = optional(string)<br>              target_dscp        = optional(string)<br>              bgp_peers = optional(list(object({<br>                ip_address          = string<br>                address_control     = optional(list(string))<br>                allowed_self_as_cnt = optional(string)<br>                annotation          = optional(string)<br>                bgp_controls = optional(object(<br>                  {<br>                    allow_self_as     = optional(bool)<br>                    as_override       = optional(bool)<br>                    dis_peer_as_check = optional(bool)<br>                    nh_self           = optional(bool)<br>                    send_com          = optional(bool)<br>                    send_ext_com      = optional(bool)<br>                  }<br>                ))<br>                alias                  = optional(string)<br>                password               = optional(string)<br>                peer_controls          = optional(list(string))<br>                private_as_control     = optional(list(string))<br>                ebgp_multihop_ttl      = optional(string)<br>                weight                 = optional(string)<br>                as_number              = optional(string)<br>                local_asn              = optional(string)<br>                local_as_number_config = optional(string)<br>                admin_state            = optional(string)<br>                route_control_profiles = optional(list(object({<br>                  direction = string<br>                  target_dn = string<br>                  }<br>                )))<br>                }<br>              )))<br>              secondary_addresses = optional(list(object(<br>                {<br>                  ip_address = string<br>                  ipv6_dad   = optional(string)<br>                }<br>              )))<br>              side_a = optional(object({<br>                ip_address         = string<br>                link_local_address = optional(string)<br>                secondary_addresses = optional(list(object(<br>                  {<br>                    ip_address = string<br>                    ipv6_dad   = optional(string)<br>                  }<br>                )))<br>                }<br>              ))<br>              side_b = optional(object(<br>                {<br>                  ip_address         = string<br>                  link_local_address = optional(string)<br>                  secondary_addresses = optional(list(object(<br>                    {<br>                      ip_address = string<br>                      ipv6_dad   = optional(string)<br>                    }<br>                  )))<br>                }<br>              ))<br>            }<br>          )))<br>          floating_svi = optional(list(object(<br>            {<br>              pod_id              = string<br>              node_id             = string<br>              ip_address          = string<br>              secondary_addresses = optional(list(string))<br>              description         = optional(string)<br>              mtu                 = optional(string)<br>              encap               = optional(string)<br>              encap_scope         = optional(string)<br>              mode                = optional(string)<br>              annotation          = optional(string)<br>              autostate           = optional(string)<br>              ipv6_dad            = optional(string)<br>              link_local_address  = optional(string)<br>              mac                 = optional(string)<br>              target_dscp         = optional(string)<br>              path_attributes = optional(list(object(<br>                {<br>                  domain_dn           = string<br>                  floating_address    = string<br>                  forged_transmit     = optional(bool)<br>                  mac_change          = optional(bool)<br>                  promiscuous_mode    = optional(bool)<br>                  secondary_addresses = optional(list(string))<br>                }<br>              )))<br>              bgp_peers = optional(list(object(<br>                {<br>                  ip_address          = string<br>                  address_control     = optional(list(string))<br>                  allowed_self_as_cnt = optional(string)<br>                  annotation          = optional(string)<br>                  bgp_controls = optional(object(<br>                    {<br>                      allow_self_as     = optional(bool)<br>                      as_override       = optional(bool)<br>                      dis_peer_as_check = optional(bool)<br>                      nh_self           = optional(bool)<br>                      send_com          = optional(bool)<br>                      send_ext_com      = optional(bool)<br>                    }<br>                  ))<br>                  alias                  = optional(string)<br>                  password               = optional(string)<br>                  peer_controls          = optional(list(string))<br>                  private_as_control     = optional(list(string))<br>                  ebgp_multihop_ttl      = optional(string)<br>                  weight                 = optional(string)<br>                  as_number              = optional(string)<br>                  local_asn              = optional(string)<br>                  local_as_number_config = optional(string)<br>                  admin_state            = optional(string)<br>                  route_control_profiles = optional(list(object({<br>                    direction = string<br>                    target_dn = string<br>                    }<br>                  )))<br>                }<br>              )))<br>            }<br>          )))<br>        }<br>      )))<br>    }<br>  ))</pre> | `[]` | no |
| <span id="input_multicast"></span> [multicast](#input\_multicast) | <pre>object(<br>    {<br>      annotation       = optional(string)<br>      address_families = optional(list(string))<br>    }<br>  )</pre> | `null` | no |
| <span id="input_nodes"></span> [nodes](#input\_nodes) <br> Simplified Block for defining nodes and associate Port, Channel or SVI interfaces | <pre>list(object(<br>    {<br>      node_id            = optional(string)<br>      pod_id             = optional(string)<br>      router_id          = optional(string)<br>      router_id_loopback = optional(string)<br>      loopback_address   = optional(string)<br>      static_routes = optional(list(object(<br>        {<br>          prefix              = string<br>          fallback_preference = optional(string)<br>          route_control       = optional(bool)<br>          track_policy        = optional(string)<br>          next_hop_addresses = optional(list(object(<br>            {<br>              next_hop_ip           = string<br>              preference            = optional(string)<br>              next_hop_profile_type = optional(string)<br>              track_member          = optional(string)<br>              track_policy          = optional(string)<br><br>            }<br>          )))<br>        }<br>      )))<br>      ospf_interface_profile = optional(object(<br>        {<br>          authentication_key    = optional(string)<br>          authentication_key_id = optional(string)<br>          authentication_type   = optional(string)<br>          ospf_interface_policy = optional(string)<br>          description           = optional(string)<br>          annotation            = optional(string)<br>        }<br>      ))<br>      bgp_peers = optional(list(object(<br>        {<br>          loopback_as_source  = optional(bool)<br>          ip_address          = optional(string)<br>          ipv6_address        = optional(string)<br>          address_control     = optional(list(string))<br>          allowed_self_as_cnt = optional(string)<br>          annotation          = optional(string)<br>          bgp_controls = optional(object(<br>            {<br>              allow_self_as     = optional(bool)<br>              as_override       = optional(bool)<br>              dis_peer_as_check = optional(bool)<br>              nh_self           = optional(bool)<br>              send_com          = optional(bool)<br>              send_ext_com      = optional(bool)<br>            }<br>          ))<br>          alias                  = optional(string)<br>          password               = optional(string)<br>          peer_controls          = optional(list(string))<br>          private_as_control     = optional(list(string))<br>          ebgp_multihop_ttl      = optional(string)<br>          weight                 = optional(string)<br>          as_number              = optional(string)<br>          local_asn              = optional(string)<br>          local_as_number_config = optional(string)<br>          admin_state            = optional(string)<br>          route_control_profiles = optional(list(object({<br>            direction = string<br>            target_dn = string<br>            }<br>          )))<br>        }<br>      )))<br>      interfaces = optional(list(object(<br>        {<br>          svi                      = optional(bool)<br>          anchor_node              = optional(string)<br>          port                     = optional(string)<br>          channel                  = optional(string)<br>          ip                       = optional(string)<br>          ipv6                     = optional(string)<br>          link_local_address       = optional(string)<br>          secondary_ip_addresses   = optional(list(string))<br>          secondary_ipv6_addresses = optional(list(string))<br>          vlan                     = optional(string)<br>          bgp_peers = optional(list(object(<br>            {<br>              ip_address          = optional(string)<br>              ipv6_address        = optional(string)<br>              address_control     = optional(list(string))<br>              allowed_self_as_cnt = optional(string)<br>              annotation          = optional(string)<br>              bgp_controls = optional(object(<br>                {<br>                  allow_self_as     = optional(bool)<br>                  as_override       = optional(bool)<br>                  dis_peer_as_check = optional(bool)<br>                  nh_self           = optional(bool)<br>                  send_com          = optional(bool)<br>                  send_ext_com      = optional(bool)<br>                }<br>              ))<br>              alias                  = optional(string)<br>              password               = optional(string)<br>              peer_controls          = optional(list(string))<br>              private_as_control     = optional(list(string))<br>              ebgp_multihop_ttl      = optional(string)<br>              weight                 = optional(string)<br>              as_number              = optional(string)<br>              local_asn              = optional(string)<br>              local_as_number_config = optional(string)<br>              admin_state            = optional(string)<br>              route_control_profiles = optional(list(object({<br>                direction = string<br>                target_dn = string<br>                }<br>              )))<br>            }<br>          )))<br>        }<br>      )))<br>  }))</pre> | <pre>[<br>  {<br>    "loopback_as_source": true<br>  }<br>]</pre> | no |
| <span id="input_ospf"></span> [ospf](#input\_ospf) <br> OSPF External Policy | <pre>object(<br>    {<br>      area_id   = optional(string)<br>      area_type = optional(string)<br>      area_cost = optional(string)<br>      area_ctrl = optional(list(string))<br>    }<br>  )</pre> | `null` | no |
| <span id="input_ospf_interface_profile"></span> [ospf\_interface\_profile](#input\_ospf\_interface\_profile) | <pre>object(<br>    {<br>      authentication_key    = optional(string)<br>      authentication_key_id = optional(string)<br>      authentication_type   = optional(string)<br>      ospf_interface_policy = optional(string)<br>      description           = optional(string)<br>      annotation            = optional(string)<br>    }<br>  )</pre> | `null` | no |
| <span id="input_route_control_for_dampening"></span> [route\_control\_for\_dampening](#input\_route\_control\_for\_dampening) | <pre>list(object(<br>    {<br>      address_family = optional(string) # choose between ipv4 and v6<br>      route_map_dn   = optional(string)<br>    }<br>  ))</pre> | `[]` | no |
| <span id="input_route_control_for_interleak_redistribution"></span> [route\_control\_for\_interleak\_redistribution](#input\_route\_control\_for\_interleak\_redistribution) | <pre>list(object(<br>    {<br>      source       = optional(string)<br>      route_map_dn = optional(string)<br>    }<br>  ))</pre> | `[]` | no |
| <span id="input_route_map_control_profiles"></span> [route\_map\_control\_profiles](#input\_route\_map\_control\_profiles) <br> Route Control Profiles | <pre>list(object(<br>    {<br>      annotation                 = optional(string)<br>      description                = optional(string)<br>      alias                      = optional(string)<br>      name                       = string<br>      route_control_profile_type = optional(string)<br>      contexts = optional(list(object(<br>        {<br>          name           = string<br>          action         = optional(string)<br>          order          = optional(string)<br>          set_rule_dn    = optional(string)<br>          match_rules_dn = optional(list(string))<br>        }<br>      )))<br>    }<br>  ))</pre> | `[]` | no |
| <span id="input_target_dscp"></span> [target\_dscp](#input\_target\_dscp) <br> The target differentiated services code point (DSCP) of the path attached to the L3 Outside object | `string` | `"unspecified"` | no |
| <span id="input_vpcs"></span> [vpcs](#input\_vpcs) <br> Simplified Block for defining VPCs | <pre>list(object(<br>    {<br>      pod_id = optional(string)<br>      nodes = optional(list(object(<br>        {<br>          node_id            = optional(string)<br>          router_id          = optional(string)<br>          router_id_loopback = optional(string)<br>          loopback_address   = optional(string)<br>        }<br>      )))<br>      interfaces = optional(list(object(<br>        {<br>          channel = optional(string)<br>          vlan    = optional(string)<br>          side_a = object(<br>            {<br>              ip                       = optional(string)<br>              ipv6                     = optional(string)<br>              link_local_address       = optional(string)<br>              secondary_ip_addresses   = optional(list(string))<br>              secondary_ipv6_addresses = optional(list(string))<br>          })<br>          side_b = object(<br>            {<br>              ip                       = optional(string)<br>              ipv6                     = optional(string)<br>              link_local_address       = optional(string)<br>              secondary_ip_addresses   = optional(list(string))<br>              secondary_ipv6_addresses = optional(list(string))<br>          })<br>          bgp_peers = optional(list(object(<br>            {<br>              ip_address          = optional(string)<br>              ipv6_address        = optional(string)<br>              address_control     = optional(list(string))<br>              allowed_self_as_cnt = optional(string)<br>              annotation          = optional(string)<br>              bgp_controls = optional(object(<br>                {<br>                  allow_self_as     = optional(bool)<br>                  as_override       = optional(bool)<br>                  dis_peer_as_check = optional(bool)<br>                  nh_self           = optional(bool)<br>                  send_com          = optional(bool)<br>                  send_ext_com      = optional(bool)<br>                }<br>              ))<br>              alias                  = optional(string)<br>              password               = optional(string)<br>              peer_controls          = optional(list(string))<br>              private_as_control     = optional(list(string))<br>              ebgp_multihop_ttl      = optional(string)<br>              weight                 = optional(string)<br>              as_number              = optional(string)<br>              local_asn              = optional(string)<br>              local_as_number_config = optional(string)<br>              admin_state            = optional(string)<br>              route_control_profiles = optional(list(object({<br>                direction = string<br>                target_dn = string<br>                }<br>              )))<br>            }<br>          )))<br>        }<br>      )))<br>      static_routes = optional(list(object(<br>        {<br>          prefix              = string<br>          fallback_preference = optional(string)<br>          route_control       = optional(bool)<br>          track_policy        = optional(string)<br>          next_hop_addresses = optional(list(object(<br>            {<br>              next_hop_ip           = string<br>              preference            = optional(string)<br>              next_hop_profile_type = optional(string)<br>              track_member          = optional(string)<br>              track_policy          = optional(string)<br><br>            }<br>          )))<br>        }<br>      )))<br>      ospf_interface_profile = optional(object(<br>        {<br>          authentication_key    = optional(string)<br>          authentication_key_id = optional(string)<br>          authentication_type   = optional(string)<br>          ospf_interface_policy = optional(string)<br>          description           = optional(string)<br>          annotation            = optional(string)<br>        }<br>      ))<br>      bgp_peers = optional(list(object(<br>        {<br>          loopback_as_source  = optional(bool)<br>          ip_address          = optional(string)<br>          ipv6_address        = optional(string)<br>          address_control     = optional(list(string))<br>          allowed_self_as_cnt = optional(string)<br>          annotation          = optional(string)<br>          bgp_controls = optional(object(<br>            {<br>              allow_self_as     = optional(bool)<br>              as_override       = optional(bool)<br>              dis_peer_as_check = optional(bool)<br>              nh_self           = optional(bool)<br>              send_com          = optional(bool)<br>              send_ext_com      = optional(bool)<br>            }<br>          ))<br>          alias                  = optional(string)<br>          password               = optional(string)<br>          peer_controls          = optional(list(string))<br>          private_as_control     = optional(list(string))<br>          ebgp_multihop_ttl      = optional(string)<br>          weight                 = optional(string)<br>          as_number              = optional(string)<br>          local_asn              = optional(string)<br>          local_as_number_config = optional(string)<br>          admin_state            = optional(string)<br>          route_control_profiles = optional(list(object({<br>            direction = string<br>            target_dn = string<br>            }<br>          )))<br>        }<br>      )))<br>    }<br><br>  ))</pre> | <pre>[<br>  {<br>    "loopback_as_source": true,<br>    "nodes": []<br>  }<br>]</pre> | no |
| <span id="input_vrf_dn"></span> [vrf\_dn](#input\_vrf\_dn) <br> Distinguished name of the vrf | `string` | `""` | no |
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTA4NzI2NTkyLDg0OTgwMTc4MiwtMTIyMj
IzOTEyMSwtNTM2MTI3NzUsLTE0MjA1OTA1NzQsLTQxNjcwOTIy
NCw4MDY4NDE5NzgsMTc0ODg2OTE5MiwtMjE0MDQyNjU2Myw4MD
c2NzU2OSwxNjg4NDk4ODI2LC0xNjQ1MTkxMTc3LDExNjU2MTEz
MDgsMTI2OTk5OTMzMiwtNTM5NjI2MTkzLDE4MDM1Nzg2NSwtMT
Y2OTIxNTYwOCwtNzYzNDE3MTk5LDE0NDM2MDQzNDNdfQ==
-->