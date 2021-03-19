<template>
    <div id="plan-mapper-wrapper">
        <select name="" v-model="floor">
            <option v-for="(f,i) in floors" :value="i" :key="i">{{f.name}} {{i}}</option>
        </select><br>
        <div id="plan-mapper-container">
            <canvas id="plan-mapper-canvas" @mousemove="showCoordinates"></canvas>
            <img usemap="#map" id="plan-mapper-img" :src="floor_image()" alt="" v-if="loaded">
            <div id="plan-mapper-infobg" v-if="clicked_area_id!==null" @click="reset_clicked">
                <div id="plan-mapper-info" @click="reset_clicked">
                    <h3>Помешение № {{clicked_group_key}}</h3>
                    <p>{{clicked_group_text}}</p>
                </div>
            </div>
        </div>
        <map name="map">
            <area href="#" v-for="(a,i) in areas" :key="i" :alt="group_title(a['group'])"
                  :title="group_title(a['group'])" :shape="a['shape']" :coords="a['coords']"
                  @click="_click(i)" @mouseenter="_mouseenter(i)" @mouseleave="_mouseleave(i)" />
        </map>
        <br>
    </div>
</template>


<script>

    import jsonfile from './assets/defaultgroups.json';

    const IMAGE_WIDTH=2763;
    const IMAGE_HEIGHT=3062;
    const IMAGE_SCREEN_WIDTH=690;
    const IMAGE_SCREEN_HEIGHT=765;
    const FLOORS=4;
    export default {
        name: 'HousePlanMapper',
        props: {
            msg: String
        },
        data: function () {
            return {
                floors: [
                    {name:'подвал',   img:require('./assets/plan0.png')},
                    {name:'1-й этаж', img:require('./assets/plan1.png')},
                    {name:'2-й этаж', img:require('./assets/plan2.png')},
                    {name:'3-й этаж', img:require('./assets/plan3.png')},
                ],
                floor:0,
                canvas:null,
                ctx:null,
                mouseX:null,
                mouseY:null,
                all_groups: jsonfile,
                all_areas: [
                    [
                        {shape:"poly", _coords:"2168, 2526, 2393, 2525, 2393, 2872, 2228, 2873, 2227, 2640, 2166, 2640", title:"233"},
                        {shape:"poly", _coords:"2228, 2163, 2572, 2163, 2572, 2505, 2504, 2505, 2504, 2253, 2228, 2253", title:"231"},
                        {shape:"poly", _coords:"1666, 2389, 1720, 2388, 1720, 2184, 1907, 2184, 1907, 2162, 1664, 2163", title:"227"},
                        {shape:"rect", _coords:"1485, 2419, 1541, 2505", title:"225"},
                        {shape:"poly", _coords:"1484, 2163, 1644, 2163, 1645, 2505, 1549, 2505, 1549, 2412, 1485, 2411", title:"224"},
                        {shape:"poly", _coords:"557, 2505, 616, 2504, 616, 2231, 966, 2231, 968, 2391, 1089, 2390, 1090, 2163, 557, 2163", title:"244"},
                        {shape:"poly", _coords:"1022, 1617, 1379, 1617, 1380, 1584, 1508, 1584, 1508, 1725, 1378, 1726, 1378, 1677, 1022, 1678", title:"250"},
                        {shape:"poly", _coords:"1124, 1485, 1220, 1484, 1221, 1412, 1358, 1413, 1359, 1608, 1124, 1609", title:"251"},
                        {shape:"poly", _coords:"1021, 1413, 1214, 1412, 1214, 1477, 1117, 1478, 1117, 1608, 1021, 1608", title:"249"},
                        {shape:"poly", _coords:"1379, 866, 1738, 866, 1739, 1422, 1508, 1423, 1508, 1576, 1380, 1576", title:"274"},
                        {shape:"rect", _coords:"1521, 845, 1639, 865", title:"273_274"},
                        {shape:"rect", _coords:"1471, 502, 1639, 845", title:"273"},
                        {shape:"rect", _coords:"1022, 866, 1358, 1405", title:"275"},
                        {shape:"rect", _coords:"1358, 1126, 1378, 1179", title:"275_274"},
                        {shape:"poly", _coords:"2517, 503, 2572, 503, 2572, 845, 2219, 845, 2219, 797, 2516, 797", title:"272"},
                        {shape:"poly", _coords:"2246, 510, 2508, 510, 2510, 791, 2225, 793, 2225, 613, 2246, 613", title:"271"},
                        {shape:"poly", _coords:"1657, 505, 1901, 505, 1901, 550, 1709, 550, 1709, 801, 2197, 801, 2198, 846, 1657, 846", title:"269"},
                        {shape:"poly", _coords:"1907, 509, 2176, 509, 2177, 793, 1717, 793, 1717, 555, 1907, 554", title:"270"},
                        {shape:"poly", _coords:"922, 368, 1838, 368, 1838, 482, 922, 483", title:"260"},
                        {shape:"poly", _coords:"175, 350, 385, 349, 385, 135, 490, 135, 489, 368, 922, 368, 922, 483, 174, 481", title:"261"},
                        {shape:"rect", _coords:"1845, 135, 2573, 481", title:"1N4"},
                        {shape:"poly", _coords:"934, 135, 1838, 135, 1838, 232, 1738, 232, 1740, 360, 934, 359", title:"267"},
                        {shape:"poly", _coords:"569, 503, 971, 503, 971, 604, 1093, 604, 1093, 845, 570, 845", title:"262"},
                        {shape:"poly", _coords:"175, 502, 548, 503, 549, 846, 174, 845, 235, 792, 515, 793, 513, 538, 235, 539, 235, 792, 176, 844", title:"263"},
                        {shape:"rect", _coords:"1666, 2394, 2208, 2504", title:"226"},
                        {shape:"rect", _coords:"1306, 2162, 1463, 2309", title:"248"},
                        {shape:"rect", _coords:"2228, 2261, 2499, 2504", title:"234"},
                        {shape:"rect", _coords:"2054, 2163, 2208, 2388", title:"230"},
                        {shape:"rect", _coords:"1913, 2163, 2047, 2389", title:"229"},
                        {shape:"rect", _coords:"1729, 2192, 1907, 2389", title:"228"},
                        {shape:"rect", _coords:"2414, 2526, 2572, 2873", title:"235"},
                        {shape:"rect", _coords:"1843, 2646, 2221, 2833", title:"236"},
                        {shape:"rect", _coords:"1874, 2527, 2161, 2640", title:"232"},
                        {shape:"rect", _coords:"942, 2533, 1813, 2824", title:"237"},
                        {shape:"rect", _coords:"175, 2647, 915, 2873", title:"239"},
                        {shape:"rect", _coords:"176, 2527, 884, 2639", title:"238"},
                        {shape:"rect", _coords:"1111, 2162, 1287, 2311", title:"246"},
                        {shape:"rect", _coords:"1308, 2319, 1463, 2505", title:"247"},
                        {shape:"rect", _coords:"1112, 2319, 1286, 2505", title:"245"},
                        {shape:"rect", _coords:"974, 2397, 1084, 2505", title:"241"},
                        {shape:"rect", _coords:"626, 2239, 955, 2505", title:"242"},
                        {shape:"rect", _coords:"175, 2162, 536, 2505", title:"240"},
                        {shape:"rect", _coords:"1021, 1686, 1200, 2142", title:"256"},
                        {shape:"rect", _coords:"1206, 1951, 1373, 2142", title:"257"},
                        {shape:"rect", _coords:"1205, 1684, 1358, 1945", title:"255"},
                        {shape:"rect", _coords:"1517, 1654, 1736, 2142", title:"253"},
                        {shape:"rect", _coords:"1379, 1735, 1508, 2141", title:"254"},
                        {shape:"rect", _coords:"1516, 1429, 1737, 1647", title:"252"},
                        {shape:"rect", _coords:"1297, 502, 1449, 844", title:"276"},
                        {shape:"rect", _coords:"1115, 503, 1277, 846", title:"258"},
                        {shape:"rect", _coords:"979, 503, 1093, 597", title:"259"},
                        {shape:"rect", _coords:"243, 545, 506, 784", title:"264"},
                        {shape:"rect", _coords:"1746, 241, 1838, 359", title:"268"},
                        {shape:"rect", _coords:"496, 134, 913, 360", title:"266"},
                        {shape:"rect", _coords:"172, 133, 377, 342", title:"265"},
                    ],
                    [
                        {shape:"rect", _coords:"1904, 182, 2097, 427", title:"61"},
                        {shape:"rect", _coords:"1904, 432, 1965, 529", title:"60"},
                        {shape:"rect", _coords:"1971, 432, 2066, 543", title:"70"},
                        {shape:"rect", _coords:"2105, 182, 2280, 354", title:"62"},
                        {shape:"rect", _coords:"1536, 2208, 1699, 2330", title:"6"},
                        {shape:"rect", _coords:"1535, 2459, 1697, 2552", title:"5"},
                        {shape:"rect", _coords:"1162, 2207, 1340, 2552", title:"34"},
                        {shape:"rect", _coords:"1067, 2442, 1142, 2551", title:"32"},
                        {shape:"rect", _coords:"1066, 2339, 1142, 2434", title:"33"},
                        {shape:"rect", _coords:"978, 2340, 1060, 2551", title:"28"},
                        {shape:"rect", _coords:"1046, 2272, 1143, 2331", title:"31"},
                        {shape:"rect", _coords:"1046, 2207, 1143, 2263", title:"30"},
                        {shape:"rect", _coords:"979, 2207, 1038, 2332", title:"29"},
                        {shape:"poly", _coords:"2073, 525, 2072, 432, 2104, 433, 2106, 363, 2281, 363, 2280, 525, 2250, 525, 2250, 543, 2128, 544, 2127, 524", title:"63"},
                        {shape:"poly", _coords:"2288, 182, 2624, 183, 2625, 359, 2402, 359, 2403, 529, 2287, 528", title:"64"},
                        {shape:"poly", _coords:"2410, 367, 2624, 368, 2624, 528, 2597, 528, 2596, 542, 2478, 542, 2477, 528, 2409, 528", title:"65"},
                        {shape:"poly", _coords:"1568, 1097, 1789, 1097, 1790, 1261, 1635, 1259, 1569, 1190", title:"76"},
                        {shape:"poly", _coords:"1718, 549, 1728, 548, 1728, 539, 1860, 539, 1860, 549, 1878, 549, 1878, 625, 1718, 625", title:"57"},
                        {shape:"poly", _coords:"2400, 2571, 2624, 2571, 2624, 2745, 2443, 2747, 2442, 2663, 2398, 2663", title:"77"},
                        {shape:"poly", _coords:"2006, 2289, 2007, 2207, 2260, 2208, 2260, 2551, 2251, 2551, 2250, 2566, 2116, 2566, 2116, 2553", title:"12"},
                        {shape:"rect", _coords:"1902, 2208, 2000, 2285", title:"11"},
                        {shape:"rect", _coords:"1718, 2208, 1902, 2400", title:"10"},
                        {shape:"poly", _coords:"1908, 2292, 2000, 2291, 2108, 2549, 2053, 2549, 2053, 2569, 2391, 2569, 2391, 2661, 1903, 2661, 1904, 2569, 1930, 2569, 1930, 2550, 1908, 2550", title:"7"},
                        {shape:"poly", _coords:"987, 2569, 1884, 2571, 1884, 2664, 1511, 2664, 1511, 2916, 1336, 2917, 1336, 2662, 987, 2663", title:"4"},
                        {shape:"poly", _coords:"611, 2443, 882, 2442, 881, 2304, 972, 2304, 971, 2552, 941, 2552, 941, 2572, 970, 2573, 969, 2662, 771, 2665, 771, 2571, 809, 2571, 809, 2552, 767, 2553, 767, 2565, 637, 2565, 637, 2552, 611, 2552", title:"20"},
                        {shape:"poly", _coords:"610, 2208, 874, 2208, 874, 2434, 680, 2435, 681, 2278, 611, 2278", title:"26"},
                        {shape:"rect", _coords:"226, 2207, 590, 2552", title:"24"},
                        {shape:"poly", _coords:"1568, 1893, 1634, 1827, 1790, 1827, 1790, 2188, 1567, 2187", title:"3"},
                        {shape:"poly", _coords:"1446, 911, 1560, 912, 1560, 1194, 1632, 1265, 1790, 1265, 1790, 1447, 1674, 1447, 1675, 1632, 1790, 1631, 1792, 1821, 1632, 1821, 1562, 1893, 1561, 2187, 1517, 2187, 1517, 2551, 1494, 2552, 1495, 2572, 1380, 2571, 1380, 2552, 1358, 2553, 1359, 2207, 1446, 2207", title:"2"},
                        {shape:"poly", _coords:"1074, 1646, 1427, 1645, 1427, 1671, 1440, 1671, 1441, 1785, 1407, 1787, 1407, 1851, 1442, 1852, 1440, 1971, 1406, 1971, 1406, 2037, 1440, 2038, 1440, 2110, 1380, 2109, 1380, 2187, 1074, 2187", title:"35"},
                        {shape:"rect", _coords:"1074, 1530, 1138, 1639", title:"36"},
                        {shape:"poly", _coords:"1073, 1087, 1155, 1087, 1155, 1019, 1238, 1018, 1240, 912, 1426, 913, 1427, 938, 1439, 939, 1440, 1063, 1426, 1063, 1426, 1123, 1440, 1123, 1440, 1248, 1426, 1248, 1426, 1468, 1406, 1468, 1407, 1639, 1144, 1639, 1145, 1528, 1269, 1528, 1268, 1463, 1075, 1463", title:"37"},
                        {shape:"poly", _coords:"1074, 911, 1232, 912, 1233, 1012, 1146, 1011, 1145, 1080, 1074, 1080", title:"38"},
                        {shape:"rect", _coords:"1682, 1455, 1790, 1623", title:"1"},
                        {shape:"poly", _coords:"2469, 739, 2508, 700, 2624, 700, 2625, 893, 2470, 893", title:"67"},
                        {shape:"poly", _coords:"2282, 700, 2428, 700, 2464, 739, 2464, 892, 2282, 893", title:"68"},
                        {shape:"poly", _coords:"2283, 548, 2624, 548, 2625, 692, 2505, 691, 2466, 731, 2428, 692, 2282, 694", title:"66"},
                        {shape:"poly", _coords:"2104, 704, 2147, 663, 2262, 662, 2262, 892, 2105, 893", title:"69"},
                        {shape:"poly", _coords:"1886, 796, 1886, 548, 2260, 548, 2260, 655, 2143, 656, 2097, 701, 2096, 893, 2012, 893, 2013, 795", title:"71"},
                        {shape:"poly", _coords:"1366, 529, 1367, 453, 1519, 453, 1517, 528, 1496, 528, 1496, 549, 1517, 548, 1517, 911, 1560, 911, 1446, 911, 1446, 893, 1436, 893, 1437, 858, 1445, 859, 1447, 637, 1357, 636, 1359, 549, 1387, 549, 1387, 529", title:"39"},
                        {shape:"rect", _coords:"882, 798, 968, 893", title:"43"},
                        {shape:"poly", _coords:"883, 547, 1008, 547, 1008, 533, 1144, 534, 1142, 643, 968, 643, 968, 791, 883, 791", title:"52"},
                        {shape:"poly", _coords:"718, 674, 860, 674, 860, 892, 764, 892, 764, 737, 718, 737", title:"45"},
                        {shape:"poly", _coords:"610, 675, 709, 675, 710, 745, 758, 744, 758, 891, 610, 892", title:"46"},
                        {shape:"poly", _coords:"440, 442, 638, 443, 640, 421, 808, 422, 808, 529, 740, 530, 741, 549, 861, 549, 861, 666, 611, 665, 610, 550, 618, 549, 617, 528, 442, 528", title:"44"},
                        {shape:"poly", _coords:"1646, 529, 1646, 437, 1693, 391, 1690, 183, 1884, 183, 1884, 528", title:"59"},
                        {shape:"poly", _coords:"1527, 183, 1686, 183, 1687, 387, 1640, 435, 1526, 434", title:"55"},
                        {shape:"poly", _coords:"1145, 181, 1361, 183, 1361, 426, 1215, 426, 1143, 327", title:"54"},
                        {shape:"poly", _coords:"989, 183, 1137, 182, 1137, 331, 1211, 432, 1360, 431, 1360, 528, 990, 529", title:"53"},
                        {shape:"poly", _coords:"815, 181, 968, 181, 968, 528, 960, 529, 960, 543, 844, 543, 843, 527, 814, 527", title:"51"},
                        {shape:"poly", _coords:"227, 182, 420, 182, 421, 529, 377, 529, 376, 543, 258, 543, 259, 527, 228, 527", title:"48"},
                        {shape:"rect", _coords:"882, 2208, 971, 2297", title:"27"},
                        {shape:"rect", _coords:"1718, 2421, 1820, 2551", title:"9"},
                        {shape:"rect", _coords:"1841, 2408, 1903, 2551", title:"8"},
                        {shape:"rect", _coords:"2443, 2752, 2624, 2918", title:"14"},
                        {shape:"rect", _coords:"2275, 2671, 2436, 2919", title:"15"},
                        {shape:"rect", _coords:"2092, 2670, 2266, 2917", title:"16"},
                        {shape:"rect", _coords:"1904, 2671, 2085, 2918", title:"17"},
                        {shape:"rect", _coords:"1704, 2671, 1884, 2917", title:"18"},
                        {shape:"rect", _coords:"1519, 2671, 1698, 2918", title:"19"},
                        {shape:"rect", _coords:"2283, 2206, 2624, 2551", title:"13"},
                        {shape:"rect", _coords:"1568, 911, 1791, 1091", title:"75"},
                        {shape:"rect", _coords:"988, 2672, 1331, 2917", title:"22"},
                        {shape:"rect", _coords:"770, 2672, 970, 2918", title:"21"},
                        {shape:"rect", _coords:"411, 2572, 762, 2917", title:"23"},
                        {shape:"rect", _coords:"226, 2571, 402, 2917", title:"25"},
                        {shape:"rect", _coords:"974, 650, 1141, 892", title:"42"},
                        {shape:"rect", _coords:"227, 548, 591, 892", title:"47"},
                        {shape:"rect", _coords:"1163, 549, 1340, 893", title:"41"},
                        {shape:"rect", _coords:"1358, 645, 1440, 853", title:"40"},
                        {shape:"rect", _coords:"1537, 645, 1698, 893", title:"74"},
                        {shape:"rect", _coords:"1718, 631, 1879, 892", title:"73"},
                        {shape:"rect", _coords:"1537, 547, 1699, 637", title:"56"},
                        {shape:"rect", _coords:"1886, 802, 2005, 892", title:"72"},
                        {shape:"rect", _coords:"1527, 442, 1640, 528", title:"58"},
                        {shape:"rect", _coords:"1367, 182, 1518, 445", title:"78"},
                        {shape:"rect", _coords:"638, 182, 808, 414", title:"50"},
                        {shape:"rect", _coords:"441, 182, 630, 435", title:"49"},
                    ],
                    [
                        {shape:"poly", _coords:"1662, 424, 1828, 425, 1829, 510, 1804, 510, 1804, 529, 2206, 529, 2207, 558, 2226, 558, 2226, 531, 2232, 531, 2233, 522, 2325, 523, 2326, 610, 1807, 609, 1673, 520, 1672, 508, 1662, 507", title:"108"},
                        {shape:"poly", _coords:"2227, 748, 2286, 748, 2328, 707, 2569, 707, 2568, 876, 2226, 874", title:"115"},
                        {shape:"poly", _coords:"2036, 617, 2325, 617, 2325, 699, 2282, 742, 2226, 741, 2227, 713, 2206, 714, 2206, 875, 2035, 873", title:"116"},
                        {shape:"poly", _coords:"1852, 873, 2030, 873, 2030, 617, 1887, 618, 1887, 838", title:"117"},
                        {shape:"poly", _coords:"1740, 772, 1741, 874, 1841, 874, 1880, 834, 1880, 616, 1807, 617, 1806, 731, 1786, 772", title:"118"},
                        {shape:"poly", _coords:"1662, 529, 1673, 530, 1800, 613, 1800, 726, 1779, 766, 1740, 766, 1741, 655, 1662, 655", title:"121"},
                        {shape:"poly", _coords:"174, 343, 235, 423, 914, 423, 914, 432, 936, 432, 936, 423, 1099, 423, 1099, 510, 936, 510, 936, 479, 912, 479, 913, 508, 712, 508, 712, 514, 586, 513, 584, 508, 519, 510, 518, 514, 387, 514, 387, 509, 321, 509, 321, 514, 203, 514, 203, 508, 174, 509", title:"99"},
                        {shape:"poly", _coords:"1484, 530, 1527, 529, 1527, 516, 1589, 516, 1589, 530, 1643, 529, 1643, 618, 1483, 618", title:"124"},
                        {shape:"poly", _coords:"1380, 953, 1393, 953, 1393, 913, 1736, 913, 1736, 1096, 1572, 1097, 1573, 1190, 1393, 1190, 1394, 1031, 1379, 1031", title:"128"},
                        {shape:"poly", _coords:"1379, 1302, 1393, 1302, 1394, 1196, 1571, 1197, 1572, 1255, 1736, 1255, 1737, 1413, 1379, 1414", title:"129"},
                        {shape:"poly", _coords:"1393, 1422, 1737, 1422, 1737, 1627, 1394, 1626, 1393, 1606, 1380, 1606, 1379, 1494, 1393, 1494", title:"131"},
                        {shape:"poly", _coords:"1379, 1879, 1394, 1879, 1394, 1818, 1736, 1819, 1736, 1969, 1394, 1968, 1394, 1948, 1379, 1948", title:"133"},
                        {shape:"poly", _coords:"1394, 1974, 1737, 1974, 1737, 2146, 1393, 2146, 1392, 2112, 1379, 2114, 1379, 2027, 1393, 2027", title:"134"},
                        {shape:"poly", _coords:"1320, 2530, 1303, 2530, 1303, 2185, 1459, 2186, 1461, 2530, 1446, 2530, 1446, 2551, 2381, 2551, 2380, 2642, 1455, 2642, 1455, 2895, 1284, 2895, 1284, 2641, 355, 2642, 356, 2550, 1319, 2549", title:"80"},
                        {shape:"poly", _coords:"1304, 529, 1317, 529, 1317, 514, 1442, 515, 1442, 529, 1460, 529, 1460, 702, 1304, 702", title:"90"},
                        {shape:"poly", _coords:"924, 530, 955, 530, 956, 515, 1076, 515, 1075, 530, 1088, 530, 1088, 657, 914, 658, 914, 540, 914, 530", title:"94"},
                        {shape:"poly", _coords:"559, 529, 586, 529, 586, 521, 714, 521, 714, 530, 740, 529, 739, 873, 558, 873", title:"97"},
                        {shape:"poly", _coords:"174, 529, 204, 529, 203, 521, 321, 521, 321, 530, 388, 530, 388, 521, 519, 521, 519, 530, 535, 529, 535, 874, 173, 873", title:"98"},
                        {shape:"poly", _coords:"2332, 523, 2355, 524, 2356, 530, 2394, 529, 2394, 699, 2332, 698", title:"113"},
                        {shape:"poly", _coords:"2403, 528, 2417, 528, 2417, 509, 2543, 510, 2544, 530, 2569, 530, 2568, 700, 2402, 699", title:"114"},
                        {shape:"poly", _coords:"2215, 163, 2569, 163, 2568, 509, 2356, 509, 2356, 518, 2233, 517, 2233, 506, 2216, 506", title:"112"},
                        {shape:"poly", _coords:"2032, 163, 2206, 163, 2207, 507, 2184, 507, 2184, 524, 2061, 524, 2061, 508, 2033, 508", title:"111"},
                        {shape:"poly", _coords:"173, 162, 384, 162, 383, 414, 237, 414, 172, 329", title:"100"},
                        {shape:"poly", _coords:"1012, 2421, 1087, 2420, 1087, 2452, 1101, 2452, 1102, 2531, 1012, 2531", title:"157"},
                        {shape:"poly", _coords:"1463, 2651, 1828, 2651, 1828, 2832, 1842, 2832, 1842, 2897, 1461, 2896", title:"144"},
                        {shape:"poly", _coords:"738, 2650, 913, 2650, 914, 2812, 928, 2813, 928, 2872, 912, 2872, 913, 2897, 738, 2896", title:"146"},
                        {shape:"poly", _coords:"2228, 2187, 2568, 2187, 2568, 2530, 2356, 2531, 2356, 2544, 2226, 2545", title:"139_3"},
                        {shape:"poly", _coords:"1662, 2340, 1886, 2340, 1886, 2530, 1804, 2529, 1804, 2544, 1682, 2544, 1682, 2530, 1662, 2530", title:"137"},
                        {shape:"poly", _coords:"1482, 2332, 1642, 2332, 1642, 2531, 1606, 2531, 1606, 2545, 1544, 2545, 1544, 2529, 1483, 2529", title:"135"},
                        {shape:"poly", _coords:"557, 2187, 734, 2187, 736, 2531, 722, 2530, 722, 2544, 584, 2544, 585, 2531, 556, 2530", title:"151"},
                        {shape:"poly", _coords:"357, 2188, 536, 2188, 537, 2545, 393, 2545, 393, 2530, 357, 2530", title:"150"},
                        {shape:"rect", _coords:"745, 528, 913, 873", title:"95"},
                        {shape:"rect", _coords:"1105, 424, 1282, 509", title:"92"},

                        {shape:"rect", _coords:"1, 4, 1, 5", title:"93"},

                        {shape:"rect", _coords:"1290, 424, 1653, 509", title:"105"},
                        {shape:"rect", _coords:"1303, 2147, 1374, 2186", title:"81_2"},
                        {shape:"rect", _coords:"1273, 913, 1374, 2147", title:"81"},
                        {shape:"rect", _coords:"1304, 872, 1373, 912", title:"89_81"},
                        {shape:"rect", _coords:"1302, 709, 1460, 874", title:"89"},
                        {shape:"rect", _coords:"2206, 2275, 2226, 2411", title:"139_2"},
                        {shape:"rect", _coords:"1894, 2187, 2206, 2529", title:"139_1"},
                        {shape:"rect", _coords:"1662, 2186, 1886, 2334", title:"138"},
                        {shape:"rect", _coords:"192, 2529, 329, 2550", title:"149_2"},
                        {shape:"rect", _coords:"174, 2185, 350, 2531", title:"149_1"},
                        {shape:"rect", _coords:"172, 2550, 349, 2896", title:"149"},
                        {shape:"rect", _coords:"2389, 2548, 2569, 2894", title:"140"},
                        {shape:"rect", _coords:"2219, 2650, 2380, 2897", title:"141"},
                        {shape:"rect", _coords:"2037, 2651, 2212, 2897", title:"142"},
                        {shape:"rect", _coords:"1851, 2649, 2031, 2897", title:"143"},
                        {shape:"rect", _coords:"936, 2650, 1276, 2896", title:"145"},
                        {shape:"rect", _coords:"553, 2649, 730, 2897", title:"147"},
                        {shape:"rect", _coords:"356, 2649, 546, 2896", title:"148"},
                        {shape:"rect", _coords:"742, 2186, 916, 2530", title:"152"},
                        {shape:"rect", _coords:"923, 2318, 1005, 2530", title:"153"},
                        {shape:"rect", _coords:"924, 2187, 986, 2310", title:"154"},
                        {shape:"rect", _coords:"990, 2186, 1088, 2243", title:"155"},
                        {shape:"rect", _coords:"990, 2250, 1087, 2311", title:"156"},
                        {shape:"rect", _coords:"1011, 2318, 1086, 2412", title:"158"},
                        {shape:"rect", _coords:"1109, 2185, 1284, 2531", title:"79"},
                        {shape:"rect", _coords:"1484, 2186, 1643, 2323", title:"136"},
                        {shape:"rect", _coords:"1020, 1990, 1267, 2147", title:"82"},
                        {shape:"rect", _coords:"1021, 1807, 1268, 1985", title:"83"},
                        {shape:"rect", _coords:"1020, 1623, 1266, 1800", title:"84"},
                        {shape:"rect", _coords:"1394, 1635, 1737, 1809", title:"132"},
                        {shape:"rect", _coords:"1020, 1438, 1269, 1616", title:"85"},
                        {shape:"rect", _coords:"1020, 1254, 1267, 1431", title:"86"},
                        {shape:"rect", _coords:"1578, 1104, 1737, 1246", title:"130"},
                        {shape:"rect", _coords:"1022, 1071, 1267, 1248", title:"87"},
                        {shape:"rect", _coords:"1021, 912, 1267, 1065", title:"88"},
                        {shape:"rect", _coords:"1662, 824, 1733, 873", title:"119"},
                        {shape:"rect", _coords:"1662, 771, 1733, 819", title:"120"},
                        {shape:"rect", _coords:"1662, 717, 1733, 765", title:"122"},
                        {shape:"rect", _coords:"1662, 664, 1734, 711", title:"123"},
                        {shape:"rect", _coords:"1484, 793, 1642, 895", title:"127"},
                        {shape:"rect", _coords:"1564, 625, 1642, 787", title:"125"},
                        {shape:"rect", _coords:"1484, 625, 1558, 787", title:"126"},
                        {shape:"rect", _coords:"1110, 528, 1284, 873", title:"91"},
                        {shape:"rect", _coords:"922, 666, 1087, 874", title:"96"},
                        {shape:"rect", _coords:"1850, 164, 2026, 509", title:"110"},
                        {shape:"rect", _coords:"1660, 163, 1829, 415", title:"109"},
                        {shape:"rect", _coords:"1470, 163, 1654, 416", title:"107"},
                        {shape:"rect", _coords:"1290, 163, 1464, 416", title:"106"},
                        {shape:"rect", _coords:"1105, 163, 1282, 415", title:"104"},
                        {shape:"rect", _coords:"935, 163, 1099, 415", title:"103"},
                        {shape:"rect", _coords:"757, 163, 913, 415", title:"102"},
                        {shape:"rect", _coords:"390, 163, 749, 412", title:"101"},
                    ],
                    [
                        {shape:"rect", _coords:"1665, 787, 1759, 850",title:"190"},
                        {shape:"rect", _coords:"1002, 2232, 1097, 2290",title:"221"},
                        {shape:"rect", _coords:"931, 141, 1284, 375",title:"179"},
                        {shape:"rect", _coords:"1292, 142, 1461, 377",title:"181"},
                        {shape:"rect", _coords:"1467, 143, 1840, 378",title:"183"},
                        {shape:"rect", _coords:"1031, 1051, 1276, 1409",title:"166"},
                        {shape:"rect", _coords:"1031, 1422, 1275, 1595",title:"165"},
                        {shape:"rect", _coords:"1404, 873, 1745, 1394",title:"198"},
                        {shape:"rect", _coords:"1405, 1404, 1743, 1605",title:"199"},
                        {shape:"rect", _coords:"1405, 1615, 1743, 2146",title:"200"},
                        {shape:"rect", _coords:"1032, 1602, 1275, 1962",title:"164"},
                        {shape:"rect", _coords:"1034, 1973, 1274, 2145",title:"163"},
                        {shape:"rect", _coords:"1471, 2274, 1651, 2386",title:"201"},
                        {shape:"rect", _coords:"1470, 2168, 1650, 2264",title:"207"},
                        {shape:"rect", _coords:"1675, 2167, 1896, 2402",title:"206"},
                        {shape:"rect", _coords:"368, 2169, 926, 2509",title:"217"},
                        {shape:"rect", _coords:"564, 2657, 936, 2872",title:"214"},
                        {shape:"rect", _coords:"934, 2167, 994, 2290",title:"219"},
                        {shape:"rect", _coords:"1002, 2168, 1097, 2218",title:"220"},
                        {shape:"rect", _coords:"1119, 2167, 1291, 2508",title:"159"},
                        {shape:"rect", _coords:"934, 2300, 1014, 2509",title:"218"},
                        {shape:"rect", _coords:"1025, 2298, 1095, 2391",title:"223"},
                        {shape:"rect", _coords:"1315, 2170, 1461, 2507",title:"161"},
                        {shape:"rect", _coords:"1469, 2395, 1651, 2507",title:"202"},
                        {shape:"rect", _coords:"1675, 2412, 1897, 2506",title:"204"},
                        {shape:"rect", _coords:"1906, 2168, 2214, 2507",title:"205"},
                        {shape:"rect", _coords:"2226, 2167, 2576, 2507",title:"208"},
                        {shape:"rect", _coords:"1850, 141, 2220, 377",title:"184"},
                        {shape:"rect", _coords:"2230, 142, 2578, 479",title:"185"},
                        {shape:"rect", _coords:"1464, 749, 1643, 851",title:"197"},
                        {shape:"rect", _coords:"1464, 580, 1545, 738",title:"196"},
                        {shape:"rect", _coords:"1554, 579, 1640, 740",title:"195"},
                        {shape:"rect", _coords:"1663, 501, 1755, 602",title:"192"},
                        {shape:"rect", _coords:"1665, 614, 1725, 703",title:"193"},
                        {shape:"rect", _coords:"1665, 714, 1758, 777",title:"191"},
                        {shape:"rect", _coords:"1770, 714, 1865, 850",title:"189"},
                        {shape:"rect", _coords:"183, 141, 528, 480",title:"177"},
                        {shape:"rect", _coords:"537, 142, 922, 376",title:"178"},
                        {shape:"rect", _coords:"1030, 874, 1274, 1043",title:"167"},
                        {shape:"rect", _coords:"1469, 388, 2221, 479",title:"182"},
                        {shape:"poly", _coords:"182, 2510, 182, 2166, 357, 2166, 357, 2509, 330, 2508, 330, 2520, 281, 2571, 222, 2515, 208, 2515, 208, 2508",title:"216"},
                        {shape:"poly", _coords:"184, 2530, 211, 2531, 212, 2522, 223, 2522, 333, 2633, 359, 2633, 359, 2876, 184, 2875",title:"215"},
                        {shape:"rect", _coords:"1022, 2399, 1110, 2509",title:"222"},
                        {shape:"poly", _coords:"183, 501, 209, 501, 210, 486, 325, 487, 325, 501, 395, 500, 395, 487, 516, 487, 516, 501, 522, 502, 524, 849, 182, 852",title:"175"},
                        {shape:"rect", _coords:"536, 385, 1280, 479",title:"171"},
                        {shape:"rect", _coords:"557, 479, 661, 501",title:"171_173"},
                        {shape:"rect", _coords:"530, 501, 817, 640",title:"173"},
                        {shape:"rect", _coords:"1115, 501, 1279, 847",title:"170"},
                        {shape:"rect", _coords:"1299, 660, 1454, 848",title:"168"},
                        {shape:"rect", _coords:"1290, 386, 1462, 478",title:"180"},
                        {shape:"rect", _coords:"1325, 478, 1431, 501",title:"180_169"},
                        {shape:"rect", _coords:"1298, 501, 1455, 650",title:"169"},
                        {shape:"poly", _coords:"530, 850, 531, 646, 636, 646, 639, 789, 592, 849",title:"176"},
                        {shape:"poly", _coords:"603, 849, 646, 791, 644, 648, 763, 647, 763, 850",title:"174"},
                        {shape:"poly", _coords:"772, 849, 771, 647, 825, 647, 826, 501, 943, 502, 942, 487, 1058, 487, 1058, 502, 1092, 502, 1092, 850",title:"172"},
                        {shape:"poly", _coords:"1462, 502, 1512, 502, 1514, 488, 1632, 487, 1634, 502, 1643, 501, 1643, 570, 1462, 569",title:"194"},
                        {shape:"poly", _coords:"1735, 612, 1765, 613, 1762, 501, 1865, 501, 1866, 705, 1734, 706",title:"188"},
                        {shape:"poly", _coords:"1874, 850, 1874, 500, 1888, 501, 1889, 486, 2023, 488, 2024, 499, 2047, 501, 2047, 847",title:"187"},
                        {shape:"poly", _coords:"2055, 850, 2054, 501, 2084, 501, 2085, 487, 2208, 488, 2208, 503, 2270, 504, 2270, 488, 2377, 486, 2377, 501, 2440, 500, 2441, 485, 2557, 487, 2556, 502, 2578, 502, 2579, 849",title:"186"},
                        {shape:"poly", _coords:"1283, 871, 1302, 869, 1298, 848, 1379, 849, 1381, 2146, 1283, 2145",title:"162"},
                        {shape:"poly", _coords:"286, 2578, 329, 2531, 403, 2529, 402, 2516, 531, 2516, 531, 2531, 555, 2529, 556, 2876, 368, 2876, 367, 2624, 334, 2623",title:"213"},
                        {shape:"poly", _coords:"564, 2530, 603, 2530, 603, 2516, 716, 2517, 716, 2530, 788, 2530, 786, 2517, 896, 2516, 897, 2530, 1011, 2530, 1013, 2646, 563, 2647",title:"211"},
                        {shape:"poly", _coords:"943, 2653, 1054, 2654, 1112, 2711, 1110, 2875, 944, 2875",title:"212"},
                        {shape:"poly", _coords:"1021, 2530, 1492, 2531, 1493, 2516, 1565, 2516, 1566, 2646, 1524, 2645, 1466, 2708, 1466, 2874, 1121, 2874, 1122, 2707, 1059, 2646, 1022, 2646",title:"160"},
                        {shape:"poly", _coords:"1475, 2876, 1474, 2708, 1530, 2653, 1788, 2653, 1844, 2709, 1845, 2876",title:"210"},
                        {shape:"poly", _coords:"1574, 2516, 1616, 2517, 1616, 2530, 1682, 2529, 1682, 2515, 1814, 2515, 1814, 2531, 2086, 2530, 2086, 2516, 2208, 2515, 2208, 2530, 2222, 2531, 2222, 2877, 1854, 2874, 1854, 2706, 1790, 2645, 1573, 2648",title:"203"},
                        {shape:"poly", _coords:"2231, 2529, 2270, 2529, 2270, 2515, 2374, 2516, 2375, 2530, 2437, 2529, 2437, 2514, 2548, 2516, 2548, 2530, 2579, 2530, 2580, 2876, 2229, 2874",title:"209"},
                    ],
                ],
                all_mapped_areas: [
                    {}, {}, {}, {}
                ],
                clicked_area_id:null,
                clicked_area_key:null,
                clicked_group_key:null,
                clicked_group_text:null,
                hovered_area_id:null,
                hovered_area_key:null,
                hovered_group_key:null,
                loaded:false,
            }
        },
        methods: {
            floor_image: function () {
                return this.floors[this.floor]['img'];
            },
            draw_area_by_id: function(area_id, style) {
                let fillStyle = "white";
                let strokeStyle = "red";
                //console.log('area_id=', area_id, 'style=', style);
                let group_id =  this.areas[area_id]['group'];
                let status = this.all_groups[this.floor][group_id]['status'];
                //console.log('group_id=', group_id, 'style=', style, 'status=', status);
                switch (style) {
                    case 'hover':
                        fillStyle = this.color(status, true, false);
                        strokeStyle = "black";
                        break;
                    case 'lite':
                        fillStyle = this.color(status, false, true);
                        strokeStyle = "black";
                        break;
                    default:
                        fillStyle = this.color(status, false, false);
                        strokeStyle = "black";
                        break;
                }
                //console.log('fillStyle=', fillStyle,);

                if (this.areas[area_id]['shape']==='rect') {
                    let c = this.areas[area_id]['scoords'];
                    //console.log('rect', c[0], c[1], (c[2] - c[0]), (c[3] - c[1]));
                    this.ctx.beginPath();
                    this.ctx.fillStyle = fillStyle;
                    this.ctx.strokeStyle = strokeStyle;
                    this.ctx.rect(c[0], c[1], (c[2] - c[0]), (c[3] - c[1]));
                    this.ctx.stroke();
                    this.ctx.fillRect(c[0], c[1], (c[2] - c[0]), (c[3] - c[1]));
                }
                else if (this.areas[area_id]['shape']==='poly') {
                    let c = this.areas[area_id]['scoords'];
                    let l = this.areas[area_id]['scoords'].length;
                    this.ctx.beginPath();
                    this.ctx.fillStyle = fillStyle;
                    this.ctx.strokeStyle = strokeStyle;
                    this.ctx.moveTo(c[l-2], c[l-1]);
                    for (let i=0; i<l; i+=2)
                    {
                        this.ctx.lineTo(c[i], c[i+1]);
                    }
                    this.ctx.closePath();
                    this.ctx.fill();
                    this.ctx.stroke();
                }
            },
            draw_area_by_key: function(area_key, style) {
                //console.log('draw_area_by_key', area_key)
                this.draw_area_by_id(this.mapped_areas[area_key], style);
            },
            draw_group: function(group_key, style) {
                //console.log('draw_group', group_key)
                //console.log('group', this.groups[group_key])
                //console.log('group.areas', this.groups[group_key].areas)
                for (let area_key of this.groups[group_key].areas) {
                    //console.log('area_key', area_key)
                    this.draw_area_by_key(area_key, style)
                }
            },
            updateCanvas: function (){
                //console.log('updateCanvas')
                this.ctx.clearRect(0, 0, IMAGE_WIDTH, IMAGE_HEIGHT);
                //console.log('clearRect', 0, 0, IMAGE_WIDTH, IMAGE_HEIGHT);
                for (let group in this.all_groups[this.floor]) {
                    if (this.hovered_group_key == group) {
                        this.draw_group(group, 'hover')
                    }
                    else {
                        this.draw_group(group, 'normal')
                    }
                }

                // if (this.hovered_area_id !== null) {
                //     this.draw_area_by_id(this.hovered_area_id, 'hover')
                // }
            },
            cleanCanvas: function (){
                //console.log('updateCanvas')
                this.ctx.clearRect(0, 0, IMAGE_WIDTH, IMAGE_HEIGHT);
            },
            showCoordinates(e) {
                this.mouseX = e.offsetX;
                this.mouseY = e.offsetY;
            },
            _click: function(i){
                //('_click', i)
                this.clicked_area_id = i;
                this.clicked_area_key = this.all_areas[this.floor][i]['title'];
                this.clicked_group_key = this.all_areas[this.floor][i]['group'];
                this.clicked_group_text = this.all_groups[this.floor][this.clicked_group_key]['title'];
            },
            // _mouseover: function(i){
            //     //console.log('_mouseover', i)
            // },
            _mouseenter: function(i){
                //console.log('_mouseenter', i)
                this.hovered_area_id = i;
                this.hovered_area_key = this.all_areas[this.floor][i]['title'];
                this.hovered_group_key = this.all_areas[this.floor][i]['group'];
                this.updateCanvas();
            },
            _mouseleave: function(){
                //console.log('_mouseleave', i)
                this.hovered_area_id = null;
                this.hovered_area_key = null;
                this.hovered_group_key = null;
                this.updateCanvas();
            },
            color: function (status, hover, lite) {
                if (lite) {
                    switch (status) {
                        case '1': return 'rgba(255, 63, 98, 0.7)'; //Арендовано (1)(розовый)
                        case '2': return 'rgba(36, 181, 0, 0.7)'; //Свободно (2)(зеленый)
                        case '3': return 'rgba(128, 128, 128, 0.7)'; //Прочие помещения (3)(серый)
                        case '4': return 'rgba(255, 221, 0, 0.7)'; //Прочие помещения  4(жёлтый),
                        case '5': return 'rgba(0, 174, 255, 0.7)'; //Прочие помещения  5(голубой),
                        case '6': return 'rgba(204, 0, 211, 0.7)'; //Прочие помещения
                        default: return 'rgba(128, 128, 128, 0.7)'; //Прочие помещения (3)(серый)
                    }
                }
                if (hover) {
                    switch (status) {
                        case '1': return 'rgba(255, 63, 98, 0.8)'; //Арендовано (1)(розовый)
                        case '2': return 'rgba(36, 181, 0, 0.8)'; //Свободно (2)(зеленый)
                        case '3': return 'rgba(128, 128, 128, 0.8)'; //Прочие помещения (3)(серый)
                        case '4': return 'rgba(255, 221, 0, 0.8)'; //Прочие помещения  4(жёлтый),
                        case '5': return 'rgba(0, 174, 255, 0.8)'; //Прочие помещения  5(голубой),
                        case '6': return 'rgba(204, 0, 211, 0.8)'; //Прочие помещения
                        default: return 'rgba(128, 128, 128, 0.8)'; //Прочие помещения (3)(серый)
                    }
                }
                else {
                    switch (status) {
                        case '1': return 'rgba(255, 63, 98, 0.5)'; //Арендовано (1)(розовый)
                        case '2': return 'rgba(36, 181, 0, 0.5)'; //Свободно (2)(зеленый)
                        case '3': return 'rgba(128, 128, 128, 0.5)'; //Прочие помещения (3)(серый)
                        case '4': return 'rgba(255, 221, 0, 0.5)'; //Прочие помещения  4(жёлтый),
                        case '5': return 'rgba(0, 174, 255, 0.5)'; //Прочие помещения  5(голубой),
                        case '6': return 'rgba(204, 0, 211, 0.5)'; //Прочие помещения
                        default: return 'rgba(128, 128, 128, 0.5)'; //Прочие помещения (3)(серый)
                    }
                }
            },
            group_title:function(group) {
                if (this.all_groups && group) {
                    return 'Помещение № ' + group + ', ' + this.all_groups[this.floor][group]['title'];
                }
                else {
                    return '';
                }
            },
            reset_clicked: function () {
                this.clicked_area_id=null;
                this.clicked_area_key=null;
                this.clicked_group_key=null;
                this.clicked_group_text=null;
            },
            fetch_data(){
                let url = '/store/groups.json';
                let self = this;
                fetch( url, {
                    headers : {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json'
                    }
                }).then(response => response.json())
                .then( json => {
                    //console.log('fetch_data ok', json)
                    self.all_groups = json;
                    self.init_data()
                    self.init_canvas()
                }, error => {
                    console.log('fetch_data error', error)
                    self.init_data()
                    self.init_canvas()
                });
            },
            init_data: function () {
                //console.log('init_data');
                for (let floor=0; floor<FLOORS; floor++) {
                    for (let i = 0; i < this.all_areas[floor].length; i++) {
                        this.all_areas[floor][i]['scoords'] = this.all_areas[floor][i]['_coords'].split(',')
                        for (let j = 0; j < this.all_areas[floor][i]['scoords'].length; j++) {
                            this.all_areas[floor][i]['scoords'][j] = Math.round(this.all_areas[floor][i]['scoords'][j] / 4);
                        }
                        this.all_areas[floor][i]['coords'] = this.all_areas[floor][i]['scoords'].join(',');
                        this.all_mapped_areas[floor][this.all_areas[floor][i]['title']] = i;
                    }
                }
                for (let floor=0; floor<FLOORS; floor++) {
                    for (let group in this.all_groups[floor]) {
                        for (let j = 0; j < this.all_groups[floor][group]['areas'].length; j++) {
                            let area_id = this.all_groups[floor][group]['areas'][j];
                            this.all_areas[floor][this.all_mapped_areas[floor][area_id]]['group'] = group;
                        }
                    }
                }
                this.loaded = true;
            },
            init_canvas: function () {
                //console.log('init_canvas');
                this.canvas = document.getElementById('plan-mapper-canvas');
                this.canvas.width = IMAGE_SCREEN_WIDTH;
                this.canvas.height = IMAGE_SCREEN_HEIGHT;
                this.ctx = this.canvas.getContext('2d');
                this.updateCanvas();
            }
        },
        computed: {
            image_width: function () {
                return IMAGE_WIDTH/4;
            },
            image_height: function () {
                return IMAGE_HEIGHT/4;
            },
            image_screen_width: function () {
                return IMAGE_SCREEN_WIDTH;
            },
            image_screen_height: function () {
                return IMAGE_SCREEN_HEIGHT;
            },
            areas: function () {
                return this.all_areas[this.floor];
            },
            groups: function () {
                return this.all_groups[this.floor];
            },
            mapped_areas: function () {
                return this.all_mapped_areas[this.floor];
            },
        },
        watch: {
            floor: function() {
                this.cleanCanvas();
                this.updateCanvas();
            }
        },
        updated() {
            //console.log('updated')
            //this.updateCanvas();
        },
        created() {
            this.fetch_data();
        },
        mounted() {

        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    div#plan-mapper-container {
        /* overflow: hidden; */
        width:690px;
        height:765px;
        position:relative;
    }
    img#plan-mapper-img {
        width:690px;
        height:765px;
        position:relative;
    }
    canvas#plan-mapper-canvas{
        pointer-events: none;
        width:690px;
        height:765px;
        position:absolute;
        z-index: 2;
    }
    div#plan-mapper-infobg{
        top:0;
        left:0;
        width:690px;
        height:765px;
        position:absolute;
        z-index: 3;
        background-color: rgba(0,0,0,0.3);
    }
    div#plan-mapper-info{
        width:250px;
        height:150px;
        position:absolute;
        left:220px;
        top:308px;
        z-index: 4;
        background-color: #FFFFFF;
        box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }
</style>
