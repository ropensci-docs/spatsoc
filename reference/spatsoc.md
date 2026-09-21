# spatsoc

spatsoc is an R package for detecting spatial and temporal groups in GPS
relocations and measuring intragroup social dynamics. It can be used to
convert GPS relocations to gambit-of-the-group format to build
proximity-based social networks, identify nearest neighbours distances
and direction, and measure individual's position with respect to the
group centroid or group leader. See all available functions below, in
the manual or on the acommpanying documentation website at
https://docs.ropensci.org/spatsoc/.

## Details

Temporal grouping:

- [`group_times()`](https://docs.ropensci.org/spatsoc/reference/group_times.md)

Spatial grouping:

- [`group_lines()`](https://docs.ropensci.org/spatsoc/reference/group_lines.md)

- [`group_polys()`](https://docs.ropensci.org/spatsoc/reference/group_polys.md)

- [`group_pts()`](https://docs.ropensci.org/spatsoc/reference/group_pts.md)

Edge-list generation:

- [`edge_dist()`](https://docs.ropensci.org/spatsoc/reference/edge_dist.md)

- [`edge_nn()`](https://docs.ropensci.org/spatsoc/reference/edge_nn.md)

- [`edge_delay()`](https://docs.ropensci.org/spatsoc/reference/edge_delay.md)

- [`edge_alignment()`](https://docs.ropensci.org/spatsoc/reference/edge_alignment.md)

- [`edge_direction()`](https://docs.ropensci.org/spatsoc/reference/edge_direction.md)

- [`edge_zones()`](https://docs.ropensci.org/spatsoc/reference/edge_zones.md)

Social network tools:

- [`randomizations()`](https://docs.ropensci.org/spatsoc/reference/randomizations.md)

- [`get_gbi()`](https://docs.ropensci.org/spatsoc/reference/get_gbi.md)

Dyad functions

- [`dyad_id()`](https://docs.ropensci.org/spatsoc/reference/dyad_id.md)

- [`fusion_id()`](https://docs.ropensci.org/spatsoc/reference/fusion_id.md)

Centroid functions

- [`centroid_group()`](https://docs.ropensci.org/spatsoc/reference/centroid_group.md)

- [`centroid_dyad()`](https://docs.ropensci.org/spatsoc/reference/centroid_dyad.md)

- [`centroid_fusion()`](https://docs.ropensci.org/spatsoc/reference/centroid_fusion.md)

Direction functions

- [`direction_step()`](https://docs.ropensci.org/spatsoc/reference/direction_step.md)

- [`direction_to_centroid()`](https://docs.ropensci.org/spatsoc/reference/direction_to_centroid.md)

- [`direction_to_leader()`](https://docs.ropensci.org/spatsoc/reference/direction_to_leader.md)

- [`direction_group()`](https://docs.ropensci.org/spatsoc/reference/direction_group.md)

- [`direction_polarization()`](https://docs.ropensci.org/spatsoc/reference/direction_polarization.md)

Distance functions

- [`distance_to_centroid()`](https://docs.ropensci.org/spatsoc/reference/distance_to_centroid.md)

- [`distance_to_leader()`](https://docs.ropensci.org/spatsoc/reference/distance_to_leader.md)

Leadership functions

- [`leader_direction_group()`](https://docs.ropensci.org/spatsoc/reference/leader_direction_group.md)

- [`leader_edge_delay()`](https://docs.ropensci.org/spatsoc/reference/leader_edge_delay.md)

Geometry interface functions

- [`get_geometry()`](https://docs.ropensci.org/spatsoc/reference/get_geometry.md)

Build functions

- [`build_lines()`](https://docs.ropensci.org/spatsoc/reference/build_lines.md)

- [`build_polys()`](https://docs.ropensci.org/spatsoc/reference/build_polys.md)

## See also

Useful links:

- <https://docs.ropensci.org/spatsoc/>

- <https://github.com/ropensci/spatsoc>

- Report bugs at <https://github.com/ropensci/spatsoc/issues>

## Author

**Maintainer**: Alec L. Robitaille <robit.alec@gmail.com>
([ORCID](https://orcid.org/0000-0002-4706-1762))

Authors:

- Quinn Webber ([ORCID](https://orcid.org/0000-0002-0434-9360))

- Eric Vander Wal ([ORCID](https://orcid.org/0000-0002-8534-4317))
