[out:json][timeout:25];

// Define area from relation
// 3600000000+1928125=3601928125
area(3601928125)->.searchArea;

// Find all peaks inside that area
node["natural"="peak"](area.searchArea);

// Output results
out body;
>;
out skel qt;
