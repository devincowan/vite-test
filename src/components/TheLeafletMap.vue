<template>
    <div id="mapContainer"></div>
</template>
  
<script setup>
import "leaflet/dist/leaflet.css";
import L from 'leaflet'
import { onMounted } from 'vue'
import { useMapStore } from '@/stores/map'

const mapStore = useMapStore()
const Map = mapStore.mapObject

onMounted(() => {
    let map = L.map('mapContainer').setView([38.2, -96], 5);
    Map.map = map;
    Map.hucbounds = [];
    Map.popups = [];
    Map.buffer = 20;
    Map.hucselected = false;
    Map.huclayers = [];
    Map.selected_hucs = [];
    Map.reaches = {};
    Map.huc2_min = 0;
    Map.huc2_max = 7;
    Map.huc4_min = 6;
    Map.huc4_max = 10;
    Map.huc6_min = 6;
    Map.huc6_max = 10;
    Map.huc10_min = 9;
    Map.huc10_max = 14;
    Map.huc12_min = 10;
    Map.huc12_max = 18;

    Map.bbox = [99999999,
        99999999,
        -99999999,
        -99999999];


    // update_lcc_bounds(["99999999",
    //     "99999999",
    //     "-99999999",
    //     "-99999999"]);

    // Initial OSM tile layer
    L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}{r}.png', {
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> &copy; <a href="http://cartodb.com/attributions">CartoDB</a>',
        subdomains: 'abcd',
        maxZoom: 19
    }).addTo(map);

    // WMS LAYER
    let url = 'http://arcgis.cuahsi.org/arcgis/services/US_WBD/HUC_WBD/MapServer/WmsServer?'

    // HUC WMS Naming
    // --------------
    // HUC12_US: 0
    // HUC10_US: 1
    // HUC_4_US: 2
    // HUC2_US: 3
    // --------------

    // HUC 2 Layer
    let huc2 = L.tileLayer.wms(url, {
        layers: 4,
        transparent: 'true',
        format: 'image/png',
        minZoom: Map.huc2_min,
        maxZoom: Map.huc2_max
    }).addTo(map);

    // HUC 4 Layer
    let huc4 = L.tileLayer.wms(url, {
        layers: 3,
        transparent: 'true',
        format: 'image/png',
        minZoom: Map.huc4_min,
        maxZoom: Map.huc4_max
    }).addTo(map);


    // HUC 12 Layer
    let huc12 = L.tileLayer.wms(url, {
        layers: 2,
        transparent: 'true',
        format: 'image/png',
        minZoom: Map.huc12_min,
        maxZoom: Map.huc12_max
    }).addTo(map);

    // HUC 10 Layer
    let huc10 = L.tileLayer.wms(url, {
        layers: 1,
        transparent: 'true',
        format: 'image/png',
        minZoom: Map.huc10_min,
        maxZoom: Map.huc10_max
    }).addTo(map);

    // add USGS gage layer to map
    url = 'http://arcgis.cuahsi.org/arcgis/services/NHD/usgs_gages/MapServer/WmsServer?';
    let gages = L.tileLayer.wms(url, {
        layers: 0,
        transparent: 'true',
        format: 'image/png',
        minZoom: 9,
        maxZoom: 18,
    }).addTo(map);

    // layer toggling
    let mixed = {
        "HUC 2": huc2,
        "HUC 4": huc4,
        "HUC 10": huc10,
        "HUC 12": huc12,
        "USGS Gages": gages
    };

    // hide the Getting Started dialog
    // $('dialog')[0].open = false;
    //    $('dialog')[0].style['visibility'] = 'hidden';



    // /*
    //  * LEAFLET BUTTONS
    //  */

    // // Erase
    // L.easyButton('fa-eraser',
    //     function () { clearSelection(); },
    //     'clear selected features').addTo(map);

    // // Help 
    // //    let btn = '<span id=help-btn class="material-icons">info-outline</i>'
    // L.easyButton('fas fa-question', function () { toggleHelpDialog(); },
    //     { position: 'topleft' }).addTo(Map.map);

    // L.control.mousePosition({
    //     prefix: "Lat Long: ",
    //     separator: ", "
    // }).addTo(map);

    // // Menu 
    // let btn = '<div id=menu-btn><strong>MENU</strong></div>';
    // let menu = L.easyButton(btn, function () { toggleMenu(); resize_map(); },
    //     { position: 'topright' }).addTo(Map.map);
    // menu.button.style.width = '80px';

    // // Submit 
    // btn = '<div id=submit-btn><strong>SUBMIT</strong></div>';
    // let btn_submit = L.easyButton(btn,
    //     function (btn) { submit(btn); },
    //     {
    //         id: 'submit',
    //         position: 'bottomright'
    //     });
    // 
    // btn_submit.button.style.width = '150px';
    // let submit_group = L.easyBar([btn_submit],
    //     {
    //         position: 'bottomright',
    //         id: 'submit-button-group'
    //     }).addTo(Map.map);
    // // save this button so it can be accessed from other functions
    // Map.submit = submit_group; //btn_submit;

    // Layer Control
    L.control.layers(null, mixed).addTo(map);

    /*
     * LEAFLET EVENT HANDLERS
     */
    map.on("click", function (e) {
        mapClick(e);
    });

    /**
    * ADD - HUC TABLE - OPEN
    * Open dialog for adding HUC to table
    */
    // document.getElementById("add-huc").onclick = function () {

    //     // show the dialog
    //     document.getElementById('addContentDialogTemplate').style.display = "";
    // };

    /**
    * ADD - HUC TABLE -SUBMIT
    * Cancels the add HUC dialog
    */
    // document.getElementById("add-huc-submit").onclick = function () {

    //     // get the content value
    //     let huc = document.getElementById('content').value;

    //     // check for 12-digit string match (i.e. HUC 12)
    //     let re = /\d{4,12}/;
    //     let match = huc.match(re);
    //     if (match) {

    //         // add this feature to the map
    //         addFeatureByHUC(huc)

    //     } else {

    //         // display error notification
    //         let message = 'Error: HUCs must contain only 8-12 numeric characters';
    //         notify(message);
    //     }

    //     // hide the dialog
    //     document.getElementById('content').value = '';
    //     document.getElementById('addContentDialogTemplate').style.display = "none";
    // }

    /**
     * ADD - HUC TABLE - CANCEL
     * Submits the add HUC dialog and applies the result to the table
     */
    // document.getElementById("add-huc-cancel").onclick = function () {

    //     // clear and hide the dialog
    //     document.getElementById('content').value = '';
    //     document.getElementById('addContentDialogTemplate').style.display = "none";
    // }

    /**
     * BBOX - OPEN
     * showBboxDialogTemplate
     **/

    // document.getElementById("show-bbox").onclick = function () {

    //     document.getElementById('showBboxDialogTemplate').style.display = "";
    // };

    /**
    * BBOX - CLOSE 
    * Cancels the rm HUC dialog
    */
    // document.getElementById("show-bbox-close").onclick = function () {

    //     // clear and hide the dialog
    //     document.getElementById('showBboxDialogTemplate').style.display = "none";
    // }



    /**
    * REMOVE - HUC TABLE - OPEN
    * Opens dialog for removing HUC from to table
    */
    // document.getElementById("rm-huc").onclick = function () {
    //     // do nothing if the button isn't activated.
    //     if (document.getElementById('rm-huc').hasClass('mdl-button--disabled')) {
    //         return;
    //     }

    //     // check if items are selected
    //     if (document.getElementById('huc-table').find('label').hasClass('is-checked')) {
    //         // show the dialog
    //         document.getElementById('rmContentDialogTemplate').style.display = "";
    //     } else {
    //         let message = 'No rows have been selected for removal';
    //         notify(message);
    //     }
    // };


    /**
    * REMOVE - HUC TABLE - CANCEL
    * Cancels the rm HUC dialog
    */
    // document.getElementById("rm-huc-cancel").onclick = function () {

    //     // clear and hide the dialog
    //     document.getElementById('rmContentDialogTemplate').style.display = "none";
    // }

    /**
    * REMOVE - HUC TABLE - SUBMIT
    * submits selected HUCs within the table for removal
    */
    // document.getElementById("rm-huc-submit").onclick = function () {

    //     // remove items from table
    //     let table = document.getElementById("huc-table");
    //     let labels = document.getElementById('huc-table tbody tr label');
    //     for (let i = labels.length - 1; i >= 0; i--) {
    //         if (labels[i].classList.contains('is-checked')) {
    //             let hucid = document.getElementById('huc-table tbody tr td')[i * 3 + 1].innerHTML;

    //             // remove the huc from the table
    //             table.deleteRow(i);

    //             // toggle the polygon off                
    //             togglePolygon(hucid, []);

    //             // remove huc from internal list of features
    //             delete Map.hucbounds[hucid];
    //         }
    //     }

    //     // update the boundaries of the global bbox
    //     updateMapBBox();
    //     let hucs = [];
    //     for (let key in Map.hucbounds) {
    //         hucs.push(key);
    //     }
    //     getLccBounds(hucs);

    //     // clear and hide the dialog
    //     document.getElementById('rmContentDialogTemplate').style.display = "none";
    // }

    // validate the map
    let box = validate_bbox_size()
    toggle_submit_button(box.is_valid);

    // fix safari map sizing issue
    // $(window).on("resize", function () {
    //     resize_map();
    // }).trigger("resize");
})

function resize_map() {
    let pheight = document.getElementById('map').parent().height();
    let pwidth = document.getElementById('map').parent().width();
    document.getElementById("map").height(pheight).width(pwidth);
    Map.map.invalidateSize();
}

/* 
 * LEAFLET HANDLERS 
 */

function toggleHelpDialog() {
    // $('dialog')[0].style.display = '';
    // $('dialog')[0].open = true;
}

function toggleMenu() {
    let accordion = document.querySelector('#accordion');
    let panel = document.querySelector('#menu-panel');
    accordion.MaterialExtAccordion.command({ action: 'toggle', target: panel });
}

function submit() {
    document.getElementById('form-submit').submit();
}

function clearHucTable() {
    // Removes all rows from the HUC table

    let table = document.getElementById("huc-table");
    let rows = document.getElementById('huc-table tbody tr');
    for (let i = rows.length - 1; i >= 0; i--) {
        table.deleteRow(i);
    }

    // toggle the delete button
    toggle_delete_button();
}

function getGageInfo(e) {

    // TESTING GAGE INFO BOX
    // quick and dirty buffer around cursor
    // bbox = lon_min, lat_min, lon_max, lat_max
    let buf = 0.001;

    let buffered_bbox = (e.latlng.lng - buf) + ','
        + (e.latlng.lat - buf) + ','
        + (e.latlng.lng + buf) + ','
        + (e.latlng.lat + buf);
    let defaultParameters = {
        service: 'WFS',
        request: 'GetFeature',
        bbox: buffered_bbox,
        typeName: 'usgs_gages:usgs_gages_4326',
        SrsName: 'EPSG:4326',
        outputFormat: 'ESRIGEOJSON'
    };
    let root = 'https://arcgis.cuahsi.org/arcgis/services/NHD/usgs_gages/MapServer/WFSServer';
    let parameters = L.Util.extend(defaultParameters);
    let gageURL = root + L.Util.getParamString(parameters);

    let gage_meta = {};
    // $.ajax({
    //     url: gageURL,
    // async: false,
    //     success: function (response) {
    //     // check that a point was found
    //     if (response.features.length > 0) {
    // 	let atts = response.features[0].attributes;
    // 	let geom = response.features[0].geometry;
    // 	gage_meta.name = atts.STATION_NM;
    // 	gage_meta.num = atts.SITE_NO;
    // 	gage_meta.x = geom.x;
    // 	gage_meta.y = geom.y;
    //     }
    // },
    // error: function(response) {
    //     // todo:
    //     console.log(response);
    // }
    // });

    return gage_meta;

}

function mapClick(e) {

    /*
    * The event handler for map click events
    * @param {event} e - a map mouse click event
    * @returns - null
    */

    // exit early if not zoomed in enough.
    // this ensures that objects are not clicked until zoomed in
    let zoom = e.target.getZoom();
    if (zoom < Map.selectable_zoom) {
        return
    }


    // check if gage was clicked
    let gage = getGageInfo(e);

    // if a gage was selected, create a pop up and exit early.
    // we don't want to toggle HUC selection if a gage was clicked
    if (Object.keys(gage).length > 0) {
        // create map info object here

        // close all popups
        if (Map.popups.length > 0) {
            Map.map.closePopup();
        }

        // create new popup containing gage info
        L.popup().setLatLng([gage.y, gage.x])
            .setContent('<b>ID:</b> ' + gage.num + '<br>'
                + '<b>Name</b>: ' + gage.name + '<br>')
            //		             + '<b>Select</b>: <a onClick=traceUpstream("'+gage.num+'")>upstream</a>')
            .openOn(Map.map);

        // exit function without toggling HUC
        return;
    }

    // mark the huc as selected.
    // this will allow the bbox to be drawn.
    Map.hucselected = true;

    // get the latitude and longitude of the click event.
    // use this data to query ArcGIS WFS for the selected HUC object.
    let defaultParameters = {
        service: 'WFS',
        request: 'GetFeature',
        bbox: e.latlng.lat + ',' + e.latlng.lng + ',' + e.latlng.lat + ',' + e.latlng.lng,
        typeName: 'HUC_WBD:HUC12_US',
        SrsName: 'EPSG:4326'
    };
    let root = 'https://arcgis.cuahsi.org/arcgis/services/US_WBD/HUC_WBD/MapServer/WFSServer';
    let parameters = L.Util.extend(defaultParameters);
    let URL = root + L.Util.getParamString(parameters);

    console.log("map click params", defaultParameters)
    // load the map and table elements async
    toggleHucsAsync(URL, true, null);

}


function traceUpstream(usgs_gage) {

    console.log('traceUpstream --> selected gage = ' + usgs_gage);

    // clear any existing reaches from the map
    if (Map.reaches.obj != null) {
        Map.reaches.obj.clearLayers();
    }

    // query the upstream reaches via NLDI
    $.ajax({
        url: '/nldi-trace',
        type: 'GET',
        contentType: "application/json",
        data: {
            'site_provider': 'nwis',
            'site': usgs_gage
        },
        success: function (response) {


            // add the reaches to the map and replace the global reaches
            // object with the newly selected reaches.
            let reaches = L.geoJSON(response.features, { style: { color: 'green' } });
            Map.reaches.start_id = reaches._leaflet_id;
            Map.reaches.count = response.features.length;
            Map.reaches.obj = reaches;
            reaches.addTo(Map.map);


            // a list to store a single coordinate for each reach
            let centroids = [];


            // generate a list of points for each of the reaches
            response.features.forEach(function (reach) {

                // select the middle geometry feature.
                // This is a hack and should be replaced with something better
                geom_idx = Math.ceil(reach.geometry.coordinates.length / 2);

                geom_coord = reach.geometry.coordinates[geom_idx];
                centroids.push(geom_coord);
            })

            console.log('Number of reaches found = ' + centroids.length);

            // TODO: move this into a function since it's used in several places.
            // query the HUC geometry for each of the reach coordinate points
            // use this data to query ArcGIS WFS for the selected HUC object.
            centroids.forEach(function (coord) {
                let defaultParameters = {
                    service: 'WFS',
                    request: 'GetFeature',
                    bbox: coord[0] + ',' + coord[1] + ',' + coord[0] + ',' + coord[1],
                    typeName: 'HUC_WBD:HUC12_US',
                    SrsName: 'EPSG:4326'
                };
                let root = 'https://arcgis.cuahsi.org/arcgis/services/US_WBD/HUC_WBD/MapServer/WFSServer';
                let parameters = L.Util.extend(defaultParameters);
                let URL = root + L.Util.getParamString(parameters);

                //	   	// load the map and table elements async
                // todo: highlight the unique set of HUCs, do not toggle.
                //	   	toggleHucsAsync(URL, true, null);
            })
        },
        error: function (error) {
            console.log('error querying NLDI upstream: ' + error);
        }
    });
}


/*
 * HUC TABLE FUNCTIONS
 */


/*
 * Toggles the delete HUC menu button
 */
function toggle_delete_button() {
    let row_count = document.getElementById('huc-table tr').length;
    if (row_count > 0) {
        document.getElementById('rm-huc').removeClass('mdl-button--disabled')
    } else {
        document.getElementById('rm-huc').addClass('mdl-button--disabled')
    }
}

function addHucRow(huc_value) {
    // Adds new rows to the HUC table

    // check if the id already exists in the table.
    // if it does, don't add it again
    let existing_row_id = getRowIdByName(huc_value);
    if (existing_row_id != -999) {
        return;
    }

    let table = document.getElementById('huc-table');
    let rid = table.rows.length;
    let chkid = 'chkbx' + rid;
    let row = table.insertRow(rid);

    cell1 = row.insertCell(0);

    let lbl = document.createElement('label');
    lbl.className = 'mdl-checkbox mdl-js-checkbox mdl-js-ripple-effect mdl-data-table__select mdl-js-ripple-effect--ignore-events is-upgraded';
    lbl.setAttribute('for', chkid);

    let chk = document.createElement('input');
    chk.type = 'checkbox';
    chk.id = chkid;
    chk.className = 'mdl-checkbox__input';
    lbl.appendChild(chk);

    cell1.appendChild(lbl);

    let cell1 = row.insertCell(1);
    cell1.className = 'mdl-data-table__cell--non-numeric';
    cell1.innerHTML = huc_value;

    let cell2 = row.insertCell(2);
    cell2.innerHTML = 'Loading';

    // toggle the delete button
    toggle_delete_button();

    //   componentHandler.upgradeAllRegistered();
}

function rmHucRow(row_id) {
    // Removes a row from the HUC table
    if (row_id >= 0) {
        let table = document.getElementById("huc-table");
        table.deleteRow(row_id);
    }

    // toggle the delete button
    toggle_delete_button();
}

function getRowIdByName(huc_value) {
    // Gets the row id for a given huc value
    // Returns: index of the matching row, -999 if it doesn't exist in the table

    let row = document.getElementById("huc-table > tbody > tr:contains('" + huc_value + "')");
    if (row.length == 0) {
        return -999;
    } else {
        return row[0].rowIndex;
    }
}

function getRowByName(huc_value) {
    // Gets the row object for a given huc value
    // Returns: the row object or -999 if it doesn't exist

    let row = document.getElementById("huc-table > tbody > tr:contains('" + huc_value + "')");
    if (row.length == 0) {
        return -999;
    } else {
        return row[0];
    }
}


function clearSelection() {
    // Clears the selected features on the map

    for (let key in Map.hucbounds) {

        // clear the huc boundary list
        delete Map.hucbounds[key];

        // clear the polygon overlays
        Map.huclayers[key].clearLayers();
        delete Map.huclayers[key];

        // clear the hucs in the html template

    }

    // clear the HUC table
    clearHucTable();

    // update the map
    updateMapBBox();
    getLccBounds([]);

    // clear and update the HUC textbox
    document.querySelector('.mdl-textfield').MaterialTextfield.change('');

}


/**
* Queries the global bounding box for a list of hucs ids
* @param {array} hucids - HUC ids to query the bounding box for
* @returns {array} - bounding box for all hucID in the Spherical Lambert Confromal Conic SRS
*/
function getLccBounds(hucs) {

    // $.ajax({
    //     url: '/wbd/gethucbbox/lcc',
    //     type: 'GET',
    //     contentType: "text/plain; charset=UTF-8",
    //     data: { 'hucID': hucs.join(",") },
    //     success: function (response) {

    //         // save the calculated bbox in LCC coordinates
    //         let lcc_bbox = JSON.parse(response).bbox;

    //         // update the global lcc bbox that will be used to submit the subsetting job
    //         // update_lcc_bounds(lcc_bbox);

    //     },
    //     error: function (error) {
    //         console.log('error querying bounding box: ' + error);
    //     }
    // });

}

function addFeatureToMap(feature) {

    // toggle bounding box
    if (feature.hucid in Map.hucbounds) {
        // remove huc from list if it's already selected
        delete Map.hucbounds[feature.hucid];

        // add huc ID to the table
        let row_id = getRowIdByName(feature.hucid)
        rmHucRow(row_id);

    }
    else {
        // add huc to list of it's not selected
        Map.hucbounds[feature.hucid] = feature.bbox;

        // remove huc ID from the table
        addHucRow(feature.hucid);
    }
    // update the boundaries of the global bbox
    updateMapBBox();

    // retrieve the LCC coordinate for this bounding box
    hucs = [];
    for (key in Map.hucbounds) {
        hucs.push(key);
    }
    getLccBounds(hucs);
}

/**
* adds features to the map by HUC id using WFS:GetFeature
* @param {string} hucid - a single HUC ids to query
* @returns - null
*/
function addFeatureByHUC(hucid) {

    let remove = null;
    if (hucid.length < 12) {
        hucid += '*';
        filter = "<ogc:Filter><ogc:PropertyIsLike wildCard=\"*\"><ogc:PropertyName>HUC12</ogc:PropertyName><ogc:Literal>" + hucid + "</ogc:Literal></ogc:PropertyIsLike></ogc:Filter>"

        // add the huc row b/c this could take a while
        // and we want the user to know that the HUCs are
        // being queried. 
        // this huc will need to be removed since it's not 12 digits.
        addHucRow(hucid);
        remove = hucid;
    }
    else if (hucid.length == 12) {
        filter = "<ogc:Filter><ogc:PropertyIsEqualTo><ogc:PropertyName>HUC12</ogc:PropertyName><ogc:Literal>" + hucid + "</ogc:Literal></ogc:PropertyIsEqualTo></ogc:Filter>"
    }
    else {
        return;
    }

    let defaultParameters = {
        service: 'WFS',
        request: 'GetFeature',
        typeName: 'HUC_WBD:HUC12_US',
        SrsName: 'EPSG:4326',
        Filter: filter
    };
    let root = 'https://arcgis.cuahsi.org/arcgis/services/US_WBD/HUC_WBD/MapServer/WFSServer';
    let parameters = L.Util.extend(defaultParameters);
    let URL = root + L.Util.getParamString(parameters);

    // load the map and table elements async
    toggleHucsAsync(URL, false, remove);

}

async function toggleHucsAsync(url, remove_if_selected, remove) {

    // call the ArcGIS WFS asyncronously so that the webpage doesn't freeze.
    const response = await fetch(url)
    if (response.ok) {
        const text = await response.text()
        const data = await new window.DOMParser().parseFromString(text, "text/xml")
        // extract the HUC ID and Boundary from the WFS XML response.
        // only process the first element of the response since the user can only
        // select a single map element at a time.

        let selected_hucs = parseWfsXML(data);
        console.log("selected hucs from xml", selected_hucs[0].hucid)
        // TODO: use control logic below to either push or remove
        Map.selected_hucs.push(selected_hucs[0])
        // Map.huclayers.push(selected_hucs)


        for (let i = 0; i < selected_hucs.length; i++) {
            try {
                let res = selected_hucs[i];

                // toggle bounding box using the following rules:
                // 1. if the object is already selected, remove it from the 
                //    map and the table.
                // 2. if the object is not selected, add it to the map and
                //    add it to the table.

                if (res.hucid in Map.hucbounds) {
                    // remove only if explicitly specified to.
                    // this is because the "ADD" dialog should never
                    // remove features, unlike the map click event.
                    if (remove_if_selected) {
                        delete Map.hucbounds[res.hucid];
                        let row_id = getRowIdByName(res.hucid)
                        rmHucRow(row_id);
                        togglePolygon(res.hucid, res.geom);
                    }
                }
                else {
                    Map.hucbounds[res.hucid] = res.bbox;
                    // addHucRow(res.hucid);
                    togglePolygon(res.hucid, res.geom);

                    // add a 'success' message for this table entry
                    let row = getRowByName(res.hucid);
                    let elem = row.getElementsByTagName('td')[2]
                    elem.innerText = 'Loaded';
                    elem.style.color = 'green';
                }
            } catch (err) {
                // if there was an error adding the HUC,
                // add an error message in the table
                let row = getRowByName(hucid);
                let elem = row.getElementsByTagName('td')[2]
                elem.innerText = 'Error';
                elem.style.color = 'red';
            }
            // refresh page
            document.getElementById('huc-table-div').hide().show(0);
            document.getElementById('map').hide().show(0);
        }

        // update the boundaries of the global bbox.
        // this is all that we really care about when the
        // subset job is submitted.
        // retrieve the LCC coordinate for this bounding box
        let hucs = [];
        updateMapBBox();
        for (let key in Map.hucbounds) {
            hucs.push(key);
        }
        console.log(hucs)
        getLccBounds(hucs);

        // update the hucs list 
        // this is used to create a shapefile
        // that is exported along with the subset
        update_huc_ids(hucs);


        // remove the specified id
        if (remove != null) {
            let rid = getRowIdByName(remove);
            if (rid != -999) {
                rmHucRow(rid);
            }
        }

    } else {
        // if there was an error calling the ArcGIS WFS,
        // add an error message in the table
        let row = getRowByName(hucid);
        let elem = row.getElementsByTagName('td')[2]
        elem.innerText = 'Server Error';
        elem.style.color = 'red';
    }
}


/**
* Calculates and draws the bounding box on the map.
*/
function updateMapBBox() {

    // calculate global boundary
    let xmin = 9999999;
    let ymin = 9999999;
    let xmax = -9999999;
    let ymax = -9999999;
    for (let key in Map.hucbounds) {
        let bounds = Map.hucbounds[key].getBounds();
        if (bounds.getWest() < xmin) {
            xmin = bounds.getWest();
        }
        if (bounds.getSouth() < ymin) {
            ymin = bounds.getSouth();
        }
        if (bounds.getEast() > xmax) {
            xmax = bounds.getEast();
        }
        if (bounds.getNorth() > ymax) {
            ymax = bounds.getNorth();
        }
    }

    // save the map bbox
    Map.bbox = [xmin, ymin, xmax, ymax];


    // remove the bbox layer if it exists
    if ('BBOX' in Map.huclayers) {
        // remove the polygon overlay 
        Map.huclayers['BBOX'].clearLayers();
        delete Map.huclayers['BBOX'];
    }

    // redraw the bbox layer with new coordinates
    let coords = [[[xmin, ymin],
    [xmin, ymax],
    [xmax, ymax],
    [xmax, ymin],
    [xmin, ymin]]];
    let polygon = [{
        "type": "Polygon",
        "coordinates": coords
    }];

    // todo: add function to validate bbox and return back styling
    // check bbox area bounds
    let bbox = validate_bbox_size();

    // todo: create bbox validation function
    toggle_submit_button(bbox.is_valid)

    let json_polygon = L.geoJSON(polygon, { style: bbox.style });

    // save the layer
    Map.huclayers['BBOX'] = json_polygon;

    json_polygon.addTo(Map.map);

    return bbox.is_valid
}


/**
* Toggles HUC boundaries on the map, on/off.
* @param {string} hucID - ID of the huc to toggle
* @param {array} ptlist - vertices of the HUC polygon
*/
function togglePolygon(hucID, ptlist) {

    if (hucID in Map.huclayers) {
        // remove the polygon overlay 
        Map.huclayers[hucID].clearLayers();
        delete Map.huclayers[hucID];
    }
    else {
        // create polygon overlay
        let coords = [];
        let c = 0
        for (c = 0; c <= ptlist.length - 1; c += 2) {
            coords.push([parseFloat(ptlist[c]),
            parseFloat(ptlist[c + 1])
            ]);
        }
        let coordinates = [coords];
        let polygon = [{
            "type": "Polygon",
            "coordinates": coordinates
        }];
        let json_polygon = L.geoJSON(polygon, { style: { fillColor: 'blue' } });

        // save the layer
        Map.huclayers[hucID] = json_polygon;

        json_polygon.addTo(Map.map);
    }

}


/**
* Parses XML returned from a web feature service
* @param {object} xml - xml response from WFS service.
* @returns {object} - ID and bounding box rectangle of the HUC polygon
*/
function parseWfsXML(xml) {
    let response = [];
    let hucs = xml.getElementsByTagName('wfs:member');
    let llat = 100000;
    let ulat = -100000;
    let llon = 100000;
    let ulon = -100000;
    for (let hid = 0; hid < hucs.length; hid++) {

        let data = hucs[hid];

        // get geometry
        let points = data.getElementsByTagName('gml:posList')[0];
        let hucID = data.getElementsByTagName('US_WBD_HUC_WBD:HUC12')[0].innerHTML;
        let ptlist = points.innerHTML.split(' ');

        // calculate bounding box
        for (let i = 1; i < ptlist.length; i += 2) {
            let lat = ptlist[i];
            if (lat > ulat) {
                ulat = lat;
            }
            else if (lat < llat) {
                llat = lat;
            }
        }
        for (let i = 0; i < ptlist.length; i += 2) {
            let lon = ptlist[i];
            if (lon > ulon) {
                ulon = lon;
            }
            else if (lon < llon) {
                llon = lon;
            }
        }
        let bounds = L.rectangle([[ulat, ulon], [llat, llon]]);
        let r = [];
        r['hucid'] = hucID;
        r['bbox'] = bounds;
        r['geom'] = ptlist;
        response.push(r);
    }
    console.log("parse wfs", response)
    return response;
}

function update_huc_ids(huclist) {

    // convert huc array into csv string
    let hucs = huclist.join(",");

    // set the #hucs hidden field in the html template
    // using jquery
    console.log("hucids", hucs)
    // document.getElementById('hucs').val(hucs);

}

// function update_lcc_bounds(lcc_bbox) {

//      // add bounding box to input boxes
//      document.getElementById('llon').val(lcc_bbox[0]);
//      document.getElementById('llat').val(lcc_bbox[1]);
//      document.getElementById('ulon').val(lcc_bbox[2]);
//      document.getElementById('ulat').val(lcc_bbox[3]);

//      // update bounding box in menu
//      document.getElementById('td-llon-val').html(parseFloat(lcc_bbox[0]).toFixed(5));
//      document.getElementById('td-llat-val').html(parseFloat(lcc_bbox[1]).toFixed(5));
//      document.getElementById('td-ulon-val').html(parseFloat(lcc_bbox[2]).toFixed(5));
//      document.getElementById('td-ulat-val').html(parseFloat(lcc_bbox[3]).toFixed(5));
// }

function get_lcc_bounds() {
    let bnds = [document.getElementById('llon').val(),
    document.getElementById('llat').val(),
    document.getElementById('ulon').val(),
    document.getElementById('ulat').val()];
    return bnds;
}

function toggle_submit_button(is_valid) {

    if (is_valid) {
        //        // disable submit button bc nothing is selected
        //        document.getElementById('btn-subset-submit').prop( "disabled", false);
        // Map.submit.enable();
    } else {
        //        // enable submit button 
        //        document.getElementById('btn-subset-submit').prop( "disabled", true );
        // Map.submit.disable();
    }
}


/**
* Validates that size constraints for the subset bounding box
* @returns {object} - bounding box style and is_valid flag
*/
function validate_bbox_size() {

    // todo: turn the bounding box red and deactivate the submit button.
    let bbox = Map.bbox;

    let londiff = Math.abs(bbox[2] - bbox[0]);
    let latdiff = Math.abs(bbox[3] - bbox[1]);

    let sqr_deg = londiff * latdiff;


    let valid = true;
    if ((bbox.includes(99999999) || bbox.includes(-99999999))) {
        valid = false;
    }

    let style = {}
    if (sqr_deg < 4 && valid) {
        style = {
            fillColor: 'black',
            weight: 2,
            opacity: 1,
            color: 'green',
            fillOpacity: 0.01,
            lineJoin: 'round'
        };

    } else {

        style = {
            fillColor: 'black',
            weight: 2,
            opacity: 1,
            color: 'red',
            fillOpacity: 0.01,
            lineJoin: 'round'
        };
        valid = false;
    }
    return { style: style, is_valid: valid }
}


/**
 * Displays a notification message at the bottom of the screen 
 * using this #notification element in base.html
 */
function notify(message) {
    let notify = document.querySelector('#notification');
    let data = { message: message }
    notify.MaterialSnackbar.showSnackbar(data);

}
</script>
<style scoped>
#mapContainer {
    width: 100%;
    height: 100vh;
}
</style>