----------------------
File: actionzone.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides action zone information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/actionzone.php");
	
	/* get values from querystring or session */
	$zactionzoneid = $wtwconnect->getVal('actionzoneid','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid',''); /* identifies the map location */
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1'); /* allows for multiple instances of same object (example: 4 of the same chairs in a building or 2 of the same building in a community) */
	$zparentname = $wtwconnect->getVal('parentname',''); /* keeps things nested under buildings or communities and buildings nested under communities */
	
	$zresponse = array();
	$zactionzones = array();
	$zscripts = array();
	$i = 0;
	/* get scripts related to community, building, or thing by action zone (loadzone) */
	$zresults = $wtwconnect->query("
		select *
		from ".wtw_tableprefix."scripts
		where deleted=0
			and actionzoneid='".$zactionzoneid."';");
	foreach ($zresults as $zrow) {
		$zscripts[$i] = array(
			'scriptid'=> $zrow["scriptid"],
			'scriptname'=> $zrow["scriptname"],
			'scriptpath'=> $zrow["scriptpath"],
			'loaded'=>'0'
		);
		$i += 1;
	}

	/* select a single action zones related to community, building, or thing by action zone (loadzone) */
	$zresults = $wtwconnect->query("
		select a1.*,
			c1.analyticsid as communityanalyticsid,
			c1.communityname,
			c1.snapshotid as communitysnapshotid,
			case when c1.snapshotid is null then ''
				else (select filepath 
					from ".wtw_tableprefix."uploads 
					where uploadid=c1.snapshotid limit 1)
				end as communitysnapshoturl,
			b1.analyticsid as buildinganalyticsid,
			b1.buildingname,
			b1.snapshotid as buildingsnapshotid,
			case when b1.snapshotid is null then ''
				else (select filepath 
					from ".wtw_tableprefix."uploads 
					where uploadid=b1.snapshotid limit 1)
				end as buildingsnapshoturl,
			t1.analyticsid as thinganalyticsid,
			t1.thingname,
			t1.snapshotid as thingsnapshotid,
			case when t1.snapshotid is null then ''
				else (select filepath 
					from ".wtw_tableprefix."uploads 
					where uploadid=t1.snapshotid limit 1)
				end as thingsnapshoturl
		from ".wtw_tableprefix."actionzones a1
			left join ".wtw_tableprefix."communities c1
				on a1.communityid=c1.communityid
			left join ".wtw_tableprefix."buildings b1
				on a1.buildingid=b1.buildingid
			left join ".wtw_tableprefix."things t1
				on a1.thingid=t1.thingid
		where 
			a1.deleted=0
			and (c1.deleted is null
				or c1.deleted=0)
			and (b1.deleted is null
				or b1.deleted=0)
			and (t1.deleted is null
				or t1.deleted=0)
			and a1.actionzoneid='".$zactionzoneid."';");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zactionzoneid = $zrow["actionzoneid"];
		$zactionzonetype = $zrow["actionzonetype"];
		$zavataranimations = array();
		if ($zactionzonetype == "loadanimations") {
			$j = 0;
			$zresults2 = $wtwconnect->query("
				select az.*,
					aa.avatarid,
					aa.loadpriority,
					aa.animationevent,
					aa.animationfriendlyname,
					aa.animationicon,
					aa.objectfolder,
					aa.objectfile,
					aa.startframe,
					aa.endframe,
					aa.animationloop,
					aa.speedratio,
					aa.soundid,
					aa.soundpath,
					aa.soundmaxdistance
				from ".wtw_tableprefix."actionzoneanimations az
					inner join ".wtw_tableprefix."avataranimations aa
						on az.avataranimationid=aa.avataranimationid
				where az.actionzoneid='".$zactionzoneid."'
					and az.deleted=0;");
			foreach ($zresults2 as $zrow2) {
				$zavataranimations[$j] = array(
					'actionzoneanimationid'=> $zrow2["actionzoneanimationid"],
					'avataranimationid'=> $zrow2["avataranimationid"],
					'avatarid'=> $zrow2["avatarid"],
					'loadpriority'=> $zrow2["loadpriority"],
					'animationevent'=> $zrow2["animationevent"],
					'animationfriendlyname'=> $zrow2["animationfriendlyname"],
					'animationicon'=> $zrow2["animationicon"],
					'objectfolder'=> $zrow2["objectfolder"],
					'objectfile'=> $zrow2["objectfile"],
					'startframe'=> $zrow2["startframe"],
					'endframe'=> $zrow2["endframe"],
					'animationloop'=> $zrow2["animationloop"],
					'speedratio'=> $zrow2["speedratio"],
					'soundid'=> $zrow2["soundid"],
					'soundpath'=> $zrow2["soundpath"],
					'soundmaxdistance'=> $zrow2["soundmaxdistance"]
				);
				$j += 1;
			}
		}
		
		$zcommunityinfo = array(
			'communityid'=> $zrow["communityid"],
			'communityind'=> '',
			'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
			'snapshotid' => $zrow["communitysnapshotid"],
			'snapshoturl' => $zrow["communitysnapshoturl"],
			'analyticsid'=> $zrow["communityanalyticsid"]
		);
		$zbuildinginfo = array(
			'buildingid'=> $zrow["buildingid"],
			'buildingind'=> '',
			'buildingname'=> $wtwconnect->escapeHTML($zrow["buildingname"]),
			'snapshotid' => $zrow["buildingsnapshotid"],
			'snapshoturl' => $zrow["buildingsnapshoturl"],
			'analyticsid'=> $zrow["buildinganalyticsid"]
		);
		$zthinginfo = array(
			'thingid'=> $zrow["thingid"],
			'thingind'=> '',
			'thingname'=> $wtwconnect->escapeHTML($zrow["thingname"]),
			'snapshotid' => $zrow["thingsnapshotid"],
			'snapshoturl' => $zrow["thingsnapshoturl"],
			'analyticsid'=> $zrow["thinganalyticsid"]
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"]
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"]
		);
		$zalttag = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"]
		);
		$zaxis = array(
			'position'=> array(
				'x'=> $zrow["axispositionx"], 
				'y'=> $zrow["axispositiony"], 
				'z'=> $zrow["axispositionz"]),
			'rotation'=> array(
				'x'=> $zrow["axisrotationx"], 
				'y'=> $zrow["axisrotationy"], 
				'z'=> $zrow["axisrotationz"]),
			'rotateaxis'=> $zrow["rotateaxis"],
			'rotatedegrees'=> $zrow["rotatedegrees"],
			'rotatedirection'=> $zrow["rotatedirection"]
		);
		$zactionzones[$i] = array(
			'communityinfo'=> $zcommunityinfo,
			'buildinginfo'=> $zbuildinginfo,
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'actionzoneid'=> $zrow["actionzoneid"], 
			'actionzoneind'=> '-1',
			'actionzonename'=> $zrow["actionzonename"], 
			'actionzonetype'=> $zrow["actionzonetype"],
			'actionzoneshape'=> $zrow["actionzoneshape"],
			'attachmoldid'=> $zrow["attachmoldid"],
			'parentactionzoneid'=> $zrow["parentactionzoneid"],
			'teleportwebid'=> $zrow["teleportwebid"],
			'teleportwebtype'=> $zrow["teleportwebtype"],
			'spawnactionzoneid'=> $zrow["spawnactionzoneid"],
			'movementtype'=> $zrow["movementtype"],
			'rotatespeed'=> $zrow["rotatespeed"],
			'value1'=> $zrow["value1"],
			'value2'=> $zrow["value2"],
			'defaulteditform'=> $zrow["defaulteditform"],
			'movementdistance'=> $zrow["movementdistance"],
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'axis'=> $zaxis,
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'inloadactionzone'=> '0',
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'avataranimations'=> $zavataranimations,
			'jsfunction'=> $zrow["jsfunction"], 
			'jsparameters'=> $zrow["jsparameters"],
			'scripts'=> $zscripts,
			'shown'=>'0',
			'status'=>'0',
			'parentname'=>$zparentname,
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['actionzones'] = $zactionzones;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-actionzone.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: actionzones.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple action zones information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/actionzones.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zthingid = $wtwconnect->getVal('thingid','');
	$zparentname = $wtwconnect->getVal('parentname','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');
	
	/* select action zones related to community, building, or thing */
	$zresults = $wtwconnect->query("
			select a1.*,
				c1.analyticsid as communityanalyticsid,
				c1.communityname,
				c1.snapshotid as communitysnapshotid,
				case when c1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=c1.snapshotid limit 1)
					end as communitysnapshoturl,
				b1.analyticsid as buildinganalyticsid,
				b1.buildingname,
				b1.snapshotid as buildingsnapshotid,
				case when b1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=b1.snapshotid limit 1)
					end as buildingsnapshoturl,
				t1.analyticsid as thinganalyticsid,
				t1.thingname,
				t1.snapshotid as thingsnapshotid,
				case when t1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=t1.snapshotid limit 1)
					end as thingsnapshoturl
			from ".wtw_tableprefix."actionzones a1 
				left join ".wtw_tableprefix."communities c1
					on a1.communityid=c1.communityid
				left join ".wtw_tableprefix."buildings b1
					on a1.buildingid=b1.buildingid
				left join ".wtw_tableprefix."things t1
					on a1.thingid=t1.thingid
			where 
				((a1.buildingid='".$zbuildingid."' 
					and a1.thingid=''
					and a1.communityid=''
					and not a1.buildingid='')
				  or (a1.communityid='".$zcommunityid."'
					and a1.thingid=''
					and a1.buildingid=''
					and not a1.communityid='')
				  or (a1.thingid='".$zthingid."'
					and a1.buildingid=''
					and a1.communityid=''
					and not a1.thingid=''))
				and a1.deleted=0
				and (c1.deleted is null
					or c1.deleted=0)
				and (b1.deleted is null
					or b1.deleted=0)
				and (t1.deleted is null
					or t1.deleted=0)
			order by a1.communityid,
				a1.buildingid,
				a1.thingid,
				a1.actionzoneid;
		");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zresponse = array();
	$zactionzones = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zactionzoneid = $zrow["actionzoneid"];
		$zactionzonetype = $zrow["actionzonetype"];
		$zavataranimations = array();
		if ($zactionzonetype == "loadanimations") {
			$j = 0;
			$zresults2 = $wtwconnect->query("
				select az.*,
					aa.avatarid,
					aa.loadpriority,
					aa.animationevent,
					aa.animationfriendlyname,
					aa.animationicon,
					aa.objectfolder,
					aa.objectfile,
					aa.startframe,
					aa.endframe,
					aa.animationloop,
					aa.speedratio,
					aa.soundid,
					aa.soundpath,
					aa.soundmaxdistance
				from ".wtw_tableprefix."actionzoneanimations az
					inner join ".wtw_tableprefix."avataranimations aa
						on az.avataranimationid=aa.avataranimationid
				where az.actionzoneid='".$zactionzoneid."'
					and az.deleted=0;");
			foreach ($zresults2 as $zrow2) {
				$zavataranimations[$j] = array(
					'actionzoneanimationid'=> $zrow2["actionzoneanimationid"],
					'avataranimationid'=> $zrow2["avataranimationid"],
					'avatarid'=> $zrow2["avatarid"],
					'loadpriority'=> $zrow2["loadpriority"],
					'animationevent'=> $zrow2["animationevent"],
					'animationfriendlyname'=> $zrow2["animationfriendlyname"],
					'animationicon'=> $zrow2["animationicon"],
					'objectfolder'=> $zrow2["objectfolder"],
					'objectfile'=> $zrow2["objectfile"],
					'startframe'=> $zrow2["startframe"],
					'endframe'=> $zrow2["endframe"],
					'animationloop'=> $zrow2["animationloop"],
					'speedratio'=> $zrow2["speedratio"],
					'soundid'=> $zrow2["soundid"],
					'soundpath'=> $zrow2["soundpath"],
					'soundmaxdistance'=> $zrow2["soundmaxdistance"]
				);
				$j += 1;
			}
		}
		
		$zscripts = array();
		$k = 0;
		/* get scripts related to community, building, or thing by action zone (loadzone) */
		$zresults3 = $wtwconnect->query("
			select *
			from ".wtw_tableprefix."scripts
			where deleted=0
				and actionzoneid='".$zactionzoneid."';");
		foreach ($zresults3 as $zrow3) {
			$zscripts[$k] = array(
				'scriptid'=> $zrow3["scriptid"],
				'scriptname'=> $zrow3["scriptname"],
				'scriptpath'=> $zrow3["scriptpath"],
				'loaded'=>'0'
			);
			$k += 1;
		}
		
		$zcommunityinfo = array(
			'communityid'=> $zrow["communityid"],
			'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
			'snapshotid' => $zrow["communitysnapshotid"],
			'snapshoturl' => $zrow["communitysnapshoturl"],
			'analyticsid'=> $zrow["communityanalyticsid"]
		);
		$zbuildinginfo = array(
			'buildingid'=> $zrow["buildingid"],
			'buildingname'=> $wtwconnect->escapeHTML($zrow["buildingname"]),
			'snapshotid' => $zrow["buildingsnapshotid"],
			'snapshoturl' => $zrow["buildingsnapshoturl"],
			'analyticsid'=> $zrow["buildinganalyticsid"]
		);
		$zthinginfo = array(
			'thingid'=> $zrow["thingid"],
			'thingname'=> $wtwconnect->escapeHTML($zrow["thingname"]),
			'snapshotid' => $zrow["thingsnapshotid"],
			'snapshoturl' => $zrow["thingsnapshoturl"],
			'analyticsid'=> $zrow["thinganalyticsid"]
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"]
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"]
		);
		$zaxis = array(
			'position'=> array(
				'x'=> $zrow["axispositionx"], 
				'y'=> $zrow["axispositiony"], 
				'z'=> $zrow["axispositionz"]),
			'rotation'=> array(
				'x'=> $zrow["axisrotationx"], 
				'y'=> $zrow["axisrotationy"], 
				'z'=> $zrow["axisrotationz"]),
			'rotateaxis'=> $zrow["rotateaxis"],
			'rotatedegrees'=> $zrow["rotatedegrees"],
			'rotatedirection'=> $zrow["rotatedirection"]
		);
		$zactionzones[$i] = array(
			'communityinfo'=> $zcommunityinfo,
			'buildinginfo'=> $zbuildinginfo,
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'actionzoneid'=> $zrow["actionzoneid"], 
			'actionzoneind'=> '-1',
			'actionzonename'=> $zrow["actionzonename"], 
			'actionzonetype'=> $zrow["actionzonetype"],
			'actionzoneshape'=> $zrow["actionzoneshape"],
			'attachmoldid'=> $zrow["attachmoldid"],
			'parentactionzoneid'=> $zrow["parentactionzoneid"],
			'teleportwebid'=> $zrow["teleportwebid"],
			'teleportwebtype'=> $zrow["teleportwebtype"],
			'spawnactionzoneid'=> $zrow["spawnactionzoneid"],
			'movementtype'=> $zrow["movementtype"],
			'rotatespeed'=> $zrow["rotatespeed"],
			'value1'=> $zrow["value1"],
			'value2'=> $zrow["value2"],
			'defaulteditform'=> $zrow["defaulteditform"],
			'movementdistance'=> $zrow["movementdistance"],
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'axis'=> $zaxis,
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'inloadactionzone'=> '0',
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'avataranimations'=> $zavataranimations,
			'jsfunction'=> $zrow["jsfunction"], 
			'jsparameters'=> $zrow["jsparameters"],
			'scripts'=> $zscripts,
			'shown'=>'0',
			'status'=>'0',
			'parentname'=>$zparentname,
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['actionzones'] = $zactionzones;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-actionzones.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: actionzonesbywebid.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple action zones information by the 3D Community, Building or Thing called */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/actionzonesbywebid.php");

	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zthingid = $wtwconnect->getVal('thingid','');
	$zparentname = $wtwconnect->getVal('parentname','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');

	/* select action zones related to community, building, AND thing (included nested items like things in a building where the things inherit the building load zones) */
	$zresults = $wtwconnect->query("
			select distinct 
				a1.actionzoneid,
				a1.pastactionzoneid,
				a1.communityid,
				a1.buildingid,
				a1.thingid,
				c1.analyticsid as communityanalyticsid,
				c1.communityname,
				c1.snapshotid as communitysnapshotid,
				case when c1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=c1.snapshotid limit 1)
					end as communitysnapshoturl,
				'' as buildinganalyticsid,
				'' as buildingname,
				'' as buildingsnapshotid,
				'' as buildingsnapshoturl,
				'' as thinganalyticsid,
				'' as thingname,
				'' as thingsnapshotid,
				'' as thingsnapshoturl,
				a1.attachmoldid,
				'' as altconnectinggridid,
				a1.loadactionzoneid,
				a1.teleportwebid,
				a1.teleportwebtype,
				a1.spawnactionzoneid,
				a1.actionzonename,
				a1.actionzonetype,
				a1.actionzoneshape,
				a1.movementtype,
				a1.positionx,
				a1.positiony,
				a1.positionz,
				a1.scalingx,
				a1.scalingy,
				a1.scalingz,
				a1.rotationx,
				a1.rotationy,
				a1.rotationz,
				a1.axispositionx,
				a1.axispositiony,
				a1.axispositionz,
				a1.axisrotationx,
				a1.axisrotationy,
				a1.axisrotationz,
				a1.rotateaxis,
				a1.rotatedegrees,
				a1.rotatedirection,
				a1.rotatespeed,
				a1.value1,
				a1.value2,
				a1.defaulteditform,
				a1.movementdistance,
				a1.parentactionzoneid,
				a1.jsfunction,
				a1.jsparameters,
				a1.createdate,
				a1.createuserid,
				a1.updatedate,
				a1.updateuserid,
				a1.deleteddate,
				a1.deleteduserid,
				a1.deleted
			from ".wtw_tableprefix."actionzones a1 
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where communityid='".$zcommunityid."' 
							and (not communityid='') 
							and deleted=0) a2
					on a1.loadactionzoneid = a2.actionzoneid
					or a1.actionzoneid = a2.actionzoneid
				left join ".wtw_tableprefix."communities c1 
					on a1.communityid=c1.communityid
			where a1.deleted=0
			
			union all
			
				select distinct 
					a1.actionzoneid,
					a1.pastactionzoneid,
					a1.communityid,
					a1.buildingid,
					a1.thingid,
					'' as communityanalyticsid,
					'' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					b1.analyticsid as buildinganalyticsid,
					b1.buildingname,
					b1.snapshotid as buildingsnapshotid,
					case when b1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=b1.snapshotid limit 1)
						end as buildingsnapshoturl,
					'' as thinganalyticsid,
					'' as thingname,
					'' as thingsnapshotid,
					'' as thingsnapshoturl,
					a1.attachmoldid,
					'' as altconnectinggridid,
					a1.loadactionzoneid,
					a1.teleportwebid,
					a1.teleportwebtype,
					a1.spawnactionzoneid,
					a1.actionzonename,
					a1.actionzonetype,
					a1.actionzoneshape,
					a1.movementtype,
					a1.positionx,
					a1.positiony,
					a1.positionz,
					a1.scalingx,
					a1.scalingy,
					a1.scalingz,
					a1.rotationx,
					a1.rotationy,
					a1.rotationz,
					a1.axispositionx,
					a1.axispositiony,
					a1.axispositionz,
					a1.axisrotationx,
					a1.axisrotationy,
					a1.axisrotationz,
					a1.rotateaxis,
					a1.rotatedegrees,
					a1.rotatedirection,
					a1.rotatespeed,
					a1.value1,
					a1.value2,
					a1.defaulteditform,
					a1.movementdistance,
					a1.parentactionzoneid,
					a1.jsfunction,
					a1.jsparameters,
					a1.createdate,
					a1.createuserid,
					a1.updatedate,
					a1.updateuserid,
					a1.deleteddate,
					a1.deleteduserid,
					a1.deleted
			from ".wtw_tableprefix."actionzones a1 
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where buildingid='".$zbuildingid."' 
							and (not buildingid='') 
							and deleted=0) a2
					on a1.loadactionzoneid = a2.actionzoneid
					or a1.actionzoneid = a2.actionzoneid
				left join ".wtw_tableprefix."buildings b1 
					on a1.buildingid=b1.buildingid
			where a1.deleted=0
				
			union all

				select distinct 
					a1.actionzoneid,
					a1.pastactionzoneid,
					a1.communityid,
					a1.buildingid,
					a1.thingid,
					'' as communityanalyticsid,
					'' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					'' as buildinganalyticsid,
					'' as buildingname,
					'' as buildingsnapshotid,
					'' as buildingsnapshoturl,
					t1.analyticsid as thinganalyticsid,
					t1.thingname,
					t1.snapshotid as thingsnapshotid,
					case when t1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=t1.snapshotid limit 1)
						end as thingsnapshoturl,
					a1.attachmoldid,
					'' as altconnectinggridid,
					a1.loadactionzoneid,
					a1.teleportwebid,
					a1.teleportwebtype,
					a1.spawnactionzoneid,
					a1.actionzonename,
					a1.actionzonetype,
					a1.actionzoneshape,
					a1.movementtype,
					a1.positionx,
					a1.positiony,
					a1.positionz,
					a1.scalingx,
					a1.scalingy,
					a1.scalingz,
					a1.rotationx,
					a1.rotationy,
					a1.rotationz,
					a1.axispositionx,
					a1.axispositiony,
					a1.axispositionz,
					a1.axisrotationx,
					a1.axisrotationy,
					a1.axisrotationz,
					a1.rotateaxis,
					a1.rotatedegrees,
					a1.rotatedirection,
					a1.rotatespeed,
					a1.value1,
					a1.value2,
					a1.defaulteditform,
					a1.movementdistance,
					a1.parentactionzoneid,
					a1.jsfunction,
					a1.jsparameters,
					a1.createdate,
					a1.createuserid,
					a1.updatedate,
					a1.updateuserid,
					a1.deleteddate,
					a1.deleteduserid,
					a1.deleted
			from ".wtw_tableprefix."actionzones a1 
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where thingid='".$zthingid."' 
							and (not thingid='') 
							and deleted=0) a2
				on a1.loadactionzoneid = a2.actionzoneid
				or a1.actionzoneid = a2.actionzoneid
				left join ".wtw_tableprefix."things t1 
					on a1.thingid=t1.thingid
			where a1.deleted=0
			
			union all 
			
				select distinct 
					a1.actionzoneid,
					a1.pastactionzoneid,
					a1.communityid,
					a1.buildingid,
					a1.thingid,
					'' as communityanalyticsid,
					'' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					'' as buildinganalyticsid,
					'' as buildingname,
					'' as buildingsnapshotid,
					'' as buildingsnapshoturl,
					t1.analyticsid as thinganalyticsid,
					t1.thingname,
					t1.snapshotid as thingsnapshotid,
					case when t1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=t1.snapshotid limit 1)
						end as thingsnapshoturl,
					a1.attachmoldid,
					connectinggrids.connectinggridid as altconnectinggridid,
					connectinggrids.altloadactionzoneid as loadactionzoneid,
					a1.teleportwebid,
					a1.teleportwebtype,
					a1.spawnactionzoneid,
					a1.actionzonename,
					a1.actionzonetype,
					a1.actionzoneshape,
					a1.movementtype,
					a1.positionx,
					a1.positiony,
					a1.positionz,
					a1.scalingx,
					a1.scalingy,
					a1.scalingz,
					a1.rotationx,
					a1.rotationy,
					a1.rotationz,
					a1.axispositionx,
					a1.axispositiony,
					a1.axispositionz,
					a1.axisrotationx,
					a1.axisrotationy,
					a1.axisrotationz,
					a1.rotateaxis,
					a1.rotatedegrees,
					a1.rotatedirection,
					a1.rotatespeed,
					a1.value1,
					a1.value2,
					a1.defaulteditform,
					a1.movementdistance,
					a1.parentactionzoneid,
					a1.jsfunction,
					a1.jsparameters,
					a1.createdate,
					a1.createuserid,
					a1.updatedate,
					a1.updateuserid,
					a1.deleteddate,
					a1.deleteduserid,
					a1.deleted
			from (select * from ".wtw_tableprefix."connectinggrids 
						where parentwebid='".$zbuildingid."' 
							and (not parentwebid='') 
							and parentwebtype='building' 
							and childwebtype='thing' 
							and (not altloadactionzoneid='') 
							and deleted=0) connectinggrids
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not actionzonetype='loadzone') 
						and deleted=0) a1
					on connectinggrids.childwebid = a1.thingid
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not thingid='') 
							and deleted=0) a2
					on a1.loadactionzoneid = a2.actionzoneid
				and childwebid=a2.thingid
				left join ".wtw_tableprefix."things t1 
					on connectinggrids.childwebid = t1.thingid
			where a1.deleted=0

			union all 
			
				select distinct 
					a1.actionzoneid,
					a1.pastactionzoneid,
					a1.communityid,
					a1.buildingid,
					a1.thingid,
					'' as communityanalyticsid,
					'' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					'' as buildinganalyticsid,
					'' as buildingname,
					'' as buildingsnapshotid,
					'' as buildingsnapshoturl,
					t1.analyticsid as thinganalyticsid,
					t1.thingname,
					t1.snapshotid as thingsnapshotid,
					case when t1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=t1.snapshotid limit 1)
						end as thingsnapshoturl,
					a1.attachmoldid,
					connectinggrids.connectinggridid as altconnectinggridid,
					connectinggrids.altloadactionzoneid as loadactionzoneid,
					a1.teleportwebid,
					a1.teleportwebtype,
					a1.spawnactionzoneid,
					a1.actionzonename,
					a1.actionzonetype,
					a1.actionzoneshape,
					a1.movementtype,
					a1.positionx,
					a1.positiony,
					a1.positionz,
					a1.scalingx,
					a1.scalingy,
					a1.scalingz,
					a1.rotationx,
					a1.rotationy,
					a1.rotationz,
					a1.axispositionx,
					a1.axispositiony,
					a1.axispositionz,
					a1.axisrotationx,
					a1.axisrotationy,
					a1.axisrotationz,
					a1.rotateaxis,
					a1.rotatedegrees,
					a1.rotatedirection,
					a1.rotatespeed,
					a1.value1,
					a1.value2,
					a1.defaulteditform,
					a1.movementdistance,
					a1.parentactionzoneid,
					a1.jsfunction,
					a1.jsparameters,
					a1.createdate,
					a1.createuserid,
					a1.updatedate,
					a1.updateuserid,
					a1.deleteddate,
					a1.deleteduserid,
					a1.deleted
			from (select * 
					from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zcommunityid."' 
						and (not parentwebid='') 
						and parentwebtype='community' 
						and childwebtype='thing' 
						and (not altloadactionzoneid='') 
						and deleted=0) connectinggrids
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not actionzonetype='loadzone') and deleted=0) a1
					on connectinggrids.childwebid = a1.thingid
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not thingid='') and deleted=0) a2
					on a1.loadactionzoneid = a2.actionzoneid
					and childwebid=a2.thingid
				left join ".wtw_tableprefix."things t1 
					on connectinggrids.childwebid = t1.thingid
			where a1.deleted=0

			union all 
			
			select distinct 
					a1.actionzoneid,
					a1.pastactionzoneid,
					a1.communityid,
					a1.buildingid,
					a1.thingid,
					'' as communityanalyticsid,
					'' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					'' as buildinganalyticsid,
					'' as buildingname,
					'' as buildingsnapshotid,
					'' as buildingsnapshoturl,
					'' as thinganalyticsid,
					'' as thingname,
					'' as thingsnapshotid,
					'' as thingsnapshoturl,
					a1.attachmoldid,
					connectinggrids.connectinggridid as altconnectinggridid,
					connectinggrids.altloadactionzoneid as loadactionzoneid,
					a1.teleportwebid,
					a1.teleportwebtype,
					a1.spawnactionzoneid,
					a1.actionzonename,
					a1.actionzonetype,
					a1.actionzoneshape,
					a1.movementtype,
					a1.positionx,
					a1.positiony,
					a1.positionz,
					a1.scalingx,
					a1.scalingy,
					a1.scalingz,
					a1.rotationx,
					a1.rotationy,
					a1.rotationz,
					a1.axispositionx,
					a1.axispositiony,
					a1.axispositionz,
					a1.axisrotationx,
					a1.axisrotationy,
					a1.axisrotationz,
					a1.rotateaxis,
					a1.rotatedegrees,
					a1.rotatedirection,
					a1.rotatespeed,
					a1.value1,
					a1.value2,
					a1.defaulteditform,
					a1.movementdistance,
					a1.parentactionzoneid,
					a1.jsfunction,
					a1.jsparameters,
					a1.createdate,
					a1.createuserid,
					a1.updatedate,
					a1.updateuserid,
					a1.deleteddate,
					a1.deleteduserid,
					a1.deleted
			from (select * 
					from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zcommunityid."' 
						and (not parentwebid='') 
						and parentwebtype='community' 
						and childwebtype='building' 
						and (not altloadactionzoneid='') 
						and deleted=0) connectinggrids
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not actionzonetype='loadzone') 
							and deleted=0) a1
					on connectinggrids.childwebid = a1.thingid
				inner join (select * 
						from ".wtw_tableprefix."actionzones 
						where (not thingid='') 
							and deleted=0) a2
					on a1.loadactionzoneid = a2.actionzoneid
					and childwebid=a2.thingid
			where a1.deleted=0
			
			order by 
				loadactionzoneid,actionzoneid; 
		");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);	

	$i = 0;
	$zresponse = array();
	$zactionzones = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zactionzoneid = $zrow["actionzoneid"];
		$zactionzonetype = $zrow["actionzonetype"];
		$zavataranimations = array();
		if ($zactionzonetype == "loadanimations") {
			$j = 0;
			$zresults2 = $wtwconnect->query("
				select az.*,
					aa.avatarid,
					aa.loadpriority,
					aa.animationevent,
					aa.animationfriendlyname,
					aa.animationicon,
					aa.objectfolder,
					aa.objectfile,
					aa.startframe,
					aa.endframe,
					aa.animationloop,
					aa.speedratio,
					aa.soundid,
					aa.soundpath,
					aa.soundmaxdistance
				from ".wtw_tableprefix."actionzoneanimations az
					inner join ".wtw_tableprefix."avataranimations aa
						on az.avataranimationid=aa.avataranimationid
				where az.actionzoneid='".$zactionzoneid."'
					and az.deleted=0;");
			foreach ($zresults2 as $zrow2) {
				$zavataranimations[$j] = array(
					'actionzoneanimationid'=> $zrow2["actionzoneanimationid"],
					'avataranimationid'=> $zrow2["avataranimationid"],
					'avatarid'=> $zrow2["avatarid"],
					'loadpriority'=> $zrow2["loadpriority"],
					'animationevent'=> $zrow2["animationevent"],
					'animationfriendlyname'=> $zrow2["animationfriendlyname"],
					'animationicon'=> $zrow2["animationicon"],
					'objectfolder'=> $zrow2["objectfolder"],
					'objectfile'=> $zrow2["objectfile"],
					'startframe'=> $zrow2["startframe"],
					'endframe'=> $zrow2["endframe"],
					'animationloop'=> $zrow2["animationloop"],
					'speedratio'=> $zrow2["speedratio"],
					'soundid'=> $zrow2["soundid"],
					'soundpath'=> $zrow2["soundpath"],
					'soundmaxdistance'=> $zrow2["soundmaxdistance"]
				);
				$j += 1;
			}
		}
		
		$zscripts = array();
		$k = 0;
		/* get scripts related to community, building, or thing by action zone (loadzone) */
		$zresults3 = $wtwconnect->query("
			select *
			from ".wtw_tableprefix."scripts
			where deleted=0
				and actionzoneid='".$zactionzoneid."';");
		foreach ($zresults3 as $zrow3) {
			$zscripts[$k] = array(
				'scriptid'=> $zrow3["scriptid"],
				'scriptname'=> $zrow3["scriptname"],
				'scriptpath'=> $zrow3["scriptpath"],
				'loaded'=>'0'
			);
			$k += 1;
		}
		
		$zcommunityinfo = array(
			'communityid'=> $zrow["communityid"],
			'communityind'=> '',
			'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
			'snapshotid' => $zrow["communitysnapshotid"],
			'snapshoturl' => $zrow["communitysnapshoturl"],
			'analyticsid'=> $zrow["communityanalyticsid"]
		);
		$zbuildinginfo = array(
			'buildingid'=> $zrow["buildingid"],
			'buildingind'=> '',
			'buildingname'=> $wtwconnect->escapeHTML($zrow["buildingname"]),
			'snapshotid' => $zrow["buildingsnapshotid"],
			'snapshoturl' => $zrow["buildingsnapshoturl"],
			'analyticsid'=> $zrow["buildinganalyticsid"]
		);
		$zthinginfo = array(
			'thingid'=> $zrow["thingid"],
			'thingind'=> '',
			'thingname'=> $wtwconnect->escapeHTML($zrow["thingname"]),
			'snapshotid' => $zrow["thingsnapshotid"],
			'snapshoturl' => $zrow["thingsnapshoturl"],
			'analyticsid'=> $zrow["thinganalyticsid"]
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"]
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"]
		);
		$zaxis = array(
			'position'=> array(
				'x'=> $zrow["axispositionx"], 
				'y'=> $zrow["axispositiony"], 
				'z'=> $zrow["axispositionz"]),
			'rotation'=> array(
				'x'=> $zrow["axisrotationx"], 
				'y'=> $zrow["axisrotationy"], 
				'z'=> $zrow["axisrotationz"]),
			'rotateaxis'=> $zrow["rotateaxis"],
			'rotatedegrees'=> $zrow["rotatedegrees"],
			'rotatedirection'=> $zrow["rotatedirection"]
		);
		$zactionzones[$i] = array(
			'communityinfo'=> $zcommunityinfo,
			'buildinginfo'=> $zbuildinginfo,
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'actionzoneid'=> $zrow["actionzoneid"], 
			'actionzoneind'=> '-1',
			'actionzonename'=> $zrow["actionzonename"], 
			'actionzonetype'=> $zrow["actionzonetype"],
			'actionzoneshape'=> $zrow["actionzoneshape"],
			'attachmoldid'=> $zrow["attachmoldid"],
			'parentactionzoneid'=> $zrow["parentactionzoneid"],
			'teleportwebid'=> $zrow["teleportwebid"],
			'teleportwebtype'=> $zrow["teleportwebtype"],
			'spawnactionzoneid'=> $zrow["spawnactionzoneid"],
			'movementtype'=> $zrow["movementtype"],
			'rotatespeed'=> $zrow["rotatespeed"],
			'value1'=> $zrow["value1"],
			'value2'=> $zrow["value2"],
			'defaulteditform'=> $zrow["defaulteditform"],
			'movementdistance'=> $zrow["movementdistance"],
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'axis'=> $zaxis,
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'inloadactionzone'=> '0',
			'altconnectinggridid'=> $zrow["altconnectinggridid"],
			'altconnectinggridind'=> '-1',
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'avataranimations'=> $zavataranimations,
			'jsfunction'=> $zrow["jsfunction"], 
			'jsparameters'=> $zrow["jsparameters"],
			'scripts'=> $zscripts,
			'shown'=>'0',
			'status'=>'0',
			'parentname'=>$zparentname,
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['actionzones'] = $zactionzones;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-actionzonesbywebid.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: apikeys.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web aliases information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/apikeys.php");
	
	$zfunction = $wtwconnect->getPost('function','');
	$zappid = $wtwconnect->getPost('appid','');
	$zappname = $wtwconnect->getPost('appname','');
	$zhosturl = $wtwconnect->getPost('hosturl','');
	$zwtwkey = $wtwconnect->getPost('wtwkey','');
	$zwtwsecret = $wtwconnect->getPost('wtwsecret','');
	$zapikeyid = $wtwconnect->getPost('apikeyid','');
	$zapproved = $wtwconnect->getPost('approved','');

	$zappid = $wtwconnect->decode64($zappid);
	$zappname = $wtwconnect->decode64($zappname);
	$zhosturl = $wtwconnect->decode64($zhosturl);
	$zwtwkey = $wtwconnect->decode64($zwtwkey);
	$zwtwsecret = $wtwconnect->decode64($zwtwsecret);
	$zapikeyid = $wtwconnect->decode64($zapikeyid);
	
	$zreferer = $_SERVER['HTTP_REFERER'];
	
	if (substr($zhosturl, -1) == '/') {
		$zhosturl = substr($zhosturl, 0, -1);
	}
	if (substr($zreferer, -1) == '/') {
		$zreferer = substr($zreferer, 0, -1);
	}

	$serror = '';
	
	$zresponse = array(
		'serror'=>'',
		'hostid'=>'',
		'wtwkey'=>''
	);
	switch ($zfunction) {
		case "checkhost":
			echo $wtwconnect->addConnectHeader('*');
			$zapikeyid = '';
			$zappid = '';
			$zwtwsecrethash = '';
			$zdeleted = '';
			$zapproved = '';
			$zkey = "...".substr($zwtwkey, -7);
			
			$zresults = $wtwconnect->query("
				select * 
				from ".wtw_tableprefix."apikeys
				where wtwkey='".base64_encode($zwtwkey)."'
				limit 1;");
			foreach ($zresults as $zrow) {
				$zapikeyid = $zrow["apikeyid"];
				$zappid = $zrow["appid"];
				$zwtwsecrethash = $zrow["wtwsecret"];
				$zdeleted = $zrow["deleted"];
				$zapproved = $zrow["approved"];
			}
			if (!isset($zwtwsecrethash) || empty($zwtwsecrethash)) {
				/* key not found */
				$zresponse = array(
					'serror'=>'Invalid Key',
					'hostid'=>'',
					'wtwkey'=>$zkey
				);
			} else {
				if (password_verify($zwtwsecret, $zwtwsecrethash)) {
					/* secret is correct */
					$zerror = 'Valid Key';
					if ($zdeleted != 0) {
						$zerror = 'Invalid Key';
						$zapikeyid = '';
					} else if ($zapproved != 1) {
						$zerror = 'Waiting on Approval';
						$zapikeyid = '';
					}
					$zresponse = array(
						'serror'=>$zerror,
						'hostid'=>$zapikeyid,
						'wtwkey'=>$zkey
					);
				} else {
					/* could not validate secret */
					$zresponse = array(
						'serror'=>'Invalid Key',
						'hostid'=>'',
						'wtwkey'=>$zkey
					);
				}
			}
			break;
		case "hostrequest":
			echo $wtwconnect->addConnectHeader('*');
			if ($wtwconnect->hasValue($zreferer) && $wtwconnect->hasValue($zappid)) {
				$zdomainname = '';
				$zforcehttps = '1';
				$zparse = parse_url($zhosturl);
				$zdomainname = $zparse['host'];
				if (strpos(strtolower($zhosturl), 'http://') !== false) {
					$zforcehttps = '0';
				}
				
				$zfoundappid = '';
				$zapikeyid = '';
				$zdeleted = '0';
				$zapproved = '0';
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."apikeys
					where appurl='".$zreferer."'
					order by createdate
					limit 1;");
				foreach ($zresults as $zrow) {
					$zfoundappid = $zrow["appid"];
					$zapikeyid = $zrow["apikeyid"];
					$zdeleted = $zrow["deleted"];
					$zapproved = $zrow["approved"];
				}
				
				if (empty($zapikeyid)) {
					$zapikeyid = $wtwconnect->getRandomString(16,1);
					$zwtwkey = base64_encode($zwtwkey);
					
					$options = ['cost' => 11];
					$zwtwsecrethash = password_hash($zwtwsecret, PASSWORD_DEFAULT, $options);

					$wtwconnect->query("
						insert into ".wtw_tableprefix."apikeys
						   (apikeyid,
							appid,
							appname,
							appurl,
							wtwkey,
							wtwsecret,
							createdate,
							updatedate)
						   values
						   ('".$zapikeyid."',
							'".$zappid."',
							'".$zappname."',
							'".$zreferer."',
							'".$zwtwkey."',
							'".$zwtwsecrethash."',
							now(),
							now());");
					$zresponse = array(
						'serror'=>'',
						'hostid'=>$zapikeyid
					);
				} else {
					if ($zdeleted == '1') {
						$zresponse = array(
							'serror'=>'Access has been denied.',
							'hostid'=>''
						);
					} else if ($zfoundappid != $zappid) {
						$zresponse = array(
							'serror'=>'App ID could not be verified.',
							'hostid'=>''
						);
					} else if ($zapproved != '1') {
						$zresponse = array(
							'serror'=>'Access has not been approved yet.',
							'hostid'=>''
						);
					}
				}
			}
			break;
	}

	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-apikeys.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: avatar.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic avatar information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/avatar.php");

	/* get values from querystring or session */
	$zavatarid = $wtwconnect->getVal('avatarid','');
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}

	$zavatarparts = array();
	$zavataranimationdefs = array();
	$zavatargroups = array();
	$zavatargroupsall = array();

	/* pull avatar by id */

	/* pull avatar colors by id */
	$zresults = $wtwconnect->query("
		select * 
		from ".wtw_tableprefix."avatarcolors 
		where deleted=0 
			and avatarid='".$zavatarid."'
		order by avatarpart, avatarpartid;");
	$zpartind = 0;
	foreach ($zresults as $zrow) {
		$zavatarparts[$zpartind] = array(
			'avatarpartid'=> $zrow["avatarpartid"],
			'avatarpart'=> $zrow["avatarpart"],
			'diffusecolor'=> $zrow["diffusecolor"],
			'specularcolor'=> $zrow["specularcolor"],
			'emissivecolor'=> $zrow["emissivecolor"],
			'ambientcolor'=> $zrow["ambientcolor"]
		);
		$zpartind += 1;
	}

	/* pull avatar animations by id */
	$zresults = $wtwconnect->query("
		select a1.* 
			from ".wtw_tableprefix."avataranimations a1
			inner join (
				select animationevent, max(updatedate) as updatedate, max(avataranimationid) as avataranimationid 
				from ".wtw_tableprefix."avataranimations 
				where avatarid='".$zavatarid."'
					and deleted=0
					and not animationevent='onoption'
				group by animationevent) a2
			on a1.avataranimationid = a2.avataranimationid
			where a1.avatarid='".$zavatarid."' 
				and a1.deleted=0
		union
		select a3.* 
			from ".wtw_tableprefix."avataranimations a3
			inner join (
				select animationfriendlyname, max(updatedate) as updatedate, max(avataranimationid) as avataranimationid 
				from ".wtw_tableprefix."avataranimations 
				where avatarid='".$zavatarid."'
					and deleted=0
					and animationevent='onoption'
				group by animationfriendlyname) a4
			on a3.avataranimationid = a4.avataranimationid
			where a3.avatarid='".$zavatarid."' 
				and a3.deleted=0
		order by loadpriority desc, animationevent, animationfriendlyname, avataranimationid;");
	$zanimationind = 0;
	$zevent = '';
	foreach ($zresults as $zrow) {
		/* avoid duplicate animations for the same event, except for optional ones */
		if ($zrow["animationevent"] != $zevent || $zrow["animationevent"] == 'onoption') {
			$zanimationloop = true;
			if ($zrow["animationloop"] != '1') {
				$zanimationloop = false;
			}
			$zavataranimationdefs[$zanimationind] = array(
				'animationind'=> -1,
				'useravataranimationid'=> '',
				'avataranimationid'=> $zrow["avataranimationid"],
				'avatarid'=> $zrow["avatarid"],
				'loadpriority'=> (int)$zrow["loadpriority"],
				'animationevent'=> $zrow["animationevent"],
				'animationfriendlyname'=> $zrow["animationfriendlyname"],
				'animationicon'=> $zrow["animationicon"],
				'objectfolder'=> $zrow["objectfolder"],
				'objectfile'=> $zrow["objectfile"],
				'startframe'=> $zrow["startframe"],
				'endframe'=> $zrow["endframe"],
				'animationloop'=> $zanimationloop,
				'defaultspeedratio'=> $zrow["speedratio"],
				'speedratio'=> $zrow["speedratio"],
				'startweight'=> '0',
				'onanimationend'=> null,
				'walkspeed'=> '1',
				'totalframes'=> '0',
				'totalstartframe'=> '0',
				'totalendframe'=> '0',
				'soundid'=> $zrow["soundid"],
				'soundpath'=> $zrow["soundpath"],
				'soundmaxdistance'=> $zrow["soundmaxdistance"]
			);
			$zanimationind += 1;
			$zevent = $zrow["animationevent"];
		}
	}

	/* pull avatar groups by id */
	$zresults = $wtwconnect->query("
		select g1.*,
			ag1.avatarsingroupid
		from ".wtw_tableprefix."avatargroups g1 
			left join (select * from ".wtw_tableprefix."avatarsingroups where avatarid='".$zavatarid."' and deleted=0) ag1
			on ag1.avatargroupid=g1.avatargroupid
		order by g1.avatargroup, ag1.avatarsingroupid;");
	$i = 0;
	$j = 0;
	foreach ($zresults as $zrow) {
		$zavatarsingroupid = '';
		if (isset($zrow["avatarsingroupid"]) && !empty($zrow["avatarsingroupid"])) {
			$zavatarsingroupid = $zrow["avatarsingroupid"];
		}
		$zavatargroupsall[$j] = array(
			'avatarsingroupid'=> $zavatarsingroupid,
			'avatargroupid'=> $zrow["avatargroupid"],
			'avatargroup'=> $zrow["avatargroup"]
		);
		$j += 1;
		if (!empty($zavatarsingroupid)) {
			$zavatargroups[$i] = array(
				'avatarsingroupid'=> $zrow["avatarsingroupid"],
				'avatargroupid'=> $zrow["avatargroupid"],
				'avatargroup'=> $zrow["avatargroup"]
			);
			$i += 1;
		}
	}

	/* pull main avatar profile by id */
	$zresults = $wtwconnect->query("
		select a1.*,
			u1.uploadid,
			u1.filepath as snapshotpath
		from ".wtw_tableprefix."avatars a1
			left join ".wtw_tableprefix."uploads u1
				on a1.snapshotid=u1.uploadid
		where a1.deleted=0 
			and a1.avatarid='".$zavatarid."'
			and (hostuserid='".$zhostuserid."' or hostuserid='')
		order by a1.avatargroup, a1.sortorder, a1.displayname;");
	$zavatar = array();

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	function arraysort($a, $b) {
		if ($a["loadpriority"] == $b["loadpriority"]) {
			return ($a["animationfriendlyname"] > $b["animationfriendlyname"]) ? 1 : -1;
		}
		return ($a["loadpriority"] < $b["loadpriority"]) ? 1 : -1;
	}

	foreach ($zresults as $zrow) {
		if (!empty($zrow["endframe"])) {
			/* load the onwait animation if it is part of the main avatar file */
			$zavataranimationdefs[$zanimationind] = array(
				'animationind'=> -1,
				'useravataranimationid'=> '',
				'avataranimationid'=> '',
				'avatarid'=> $zrow["avatarid"],
				'loadpriority'=> 100,
				'animationevent'=> 'onwait',
				'animationfriendlyname'=> 'Default',
				'animationicon'=> '',
				'objectfolder'=> $zrow["objectfolder"],
				'objectfile'=> $zrow["objectfile"],
				'startframe'=> $zrow["startframe"],
				'endframe'=> $zrow["endframe"],
				'animationloop'=> true,
				'defaultspeedratio'=> 1,
				'speedratio'=> 1,
				'startweight'=> '0',
				'onanimationend'=> null,
				'walkspeed'=> '1',
				'totalframes'=> '0',
				'totalstartframe'=> '0',
				'totalendframe'=> '0',
				'soundid'=> '',
				'soundpath'=> '',
				'soundmaxdistance'=> 100
			);
		}
		
		usort($zavataranimationdefs, "arraysort");
		
		$zsnapshot = '';
		$zsnapshotthumbnail = '';
		
		if (file_exists(wtw_rootpath.'/content/uploads/avatars/'.$zrow["avatarid"].'/snapshots/defaultavatar.png')) {
			$zsnapshot = '/content/uploads/avatars/'.$zrow["avatarid"].'/snapshots/defaultavatar.png';
		} else {
			$zsnapshot = '/content/system/images/profilebig.png';
		}
		if (file_exists(wtw_rootpath.'/content/uploads/avatars/'.$zrow["avatarid"].'/snapshots/defaultavatarsm.png')) {
			$zsnapshotthumbnail = '/content/uploads/avatars/'.$zrow["avatarid"].'/snapshots/defaultavatarsm.png';
		} else {
			$zsnapshotthumbnail = '/content/system/images/menuprofilebig.png';
		}
		
		/* Load the avatar information for response */
		$zavatar = array(
			'avatarid'=> $zrow["avatarid"],
			'hostuserid'=> $zrow["hostuserid"],
			'versionid'=> $zrow["versionid"],
			'version'=> $zrow["version"],
			'versionorder'=> $zrow["versionorder"],
			'versiondesc'=> $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'avatargroup'=> $zrow["avatargroup"],
			'avatargroups'=> $zavatargroups,
			'avatargroupsall'=> $zavatargroupsall,
			'displayname'=> $wtwconnect->escapeHTML($zrow["displayname"]),
			'avatardescription'=> $wtwconnect->escapeHTML($zrow["avatardescription"]),
			'gender'=> $zrow["gender"],
			'objects'=> array(
				'folder'=> $zrow["objectfolder"],
				'file'=> $zrow["objectfile"],
				'startframe'=> $zrow["startframe"],
				'endframe'=> $zrow["endframe"]
			),
			'position'=> array(
				'x'=> $zrow["positionx"],
				'y'=> $zrow["positiony"],
				'z'=> $zrow["positionz"]
			),
			'scaling'=> array(
				'x'=> $zrow["scalingx"],
				'y'=> $zrow["scalingy"],
				'z'=> $zrow["scalingz"]
			),
			'rotation'=> array(
				'x'=> $zrow["rotationx"],
				'y'=> $zrow["rotationy"],
				'z'=> $zrow["rotationz"]
			),
			'snapshots'=> array(
				'full'=> $zsnapshot,
				'thumbnail'=> $zsnapshotthumbnail
			),
			'start'=> array(
				'position'=> array(
					'x'=> 0,
					'y'=> 0,
					'z'=> 0
				),
				'rotation'=> array(
					'x'=> 0,
					'y'=> 0,
					'z'=> 0
				)
			),
			'share'=> array(
				'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
				'description' => $wtwconnect->escapeHTML($zrow["description"]),
				'tags' => $wtwconnect->escapeHTML($zrow["tags"])
			),
			'sounds'=> array(
				'voice' => null
			),
			'sortorder'=> $zrow["sortorder"],
			'alttag'=> $zrow["alttag"],
			'avatarparts'=> $zavatarparts,
			'avataranimationdefs'=> $zavataranimationdefs,
			'createuserid'=> $zrow["createuserid"],
			'createdate'=> $zrow["createdate"],
			'updateuserid'=> $zrow["updateuserid"],
			'updatedate'=> $zrow["updatedate"]
		);
	}
	$zresponse['avatar'] = $zavatar;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-avatar.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: avataranimations.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic avatar animations information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/avataranimations.php");

	$i = 0;
	$zavataranimations = array();

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	/* get avatar animations */
	$zresults = $wtwconnect->query("
		select a.*
		from ".wtw_tableprefix."avataranimations a
		where a.deleted=0
		order by a.loadpriority desc, a.animationevent, a.animationfriendlyname, a.avataranimationid;");
	foreach ($zresults as $zrow) {
		$zavataranimations[$i] = array(
			'animationind'=> $i,
			'avataranimationid'=> $zrow["avataranimationid"],
			'animationevent'=> $zrow["animationevent"],
			'animationfriendlyname'=> $zrow["animationfriendlyname"],
			'loadpriority'=> $zrow["loadpriority"],
			'animationicon'=> $zrow["animationicon"],
			'speedratio'=> $zrow["speedratio"],
			'objectfolder'=> $zrow["objectfolder"],
			'objectfile'=> $zrow["objectfile"],
			'startframe'=> $zrow["startframe"],
			'endframe'=> $zrow["endframe"],
			'soundid'=> $zrow["soundid"],
			'soundpath'=> $zrow["soundpath"],
			'soundmaxdistance'=> $zrow["soundmaxdistance"]
		);
		$i += 1;
	}

	$zresponse['avataranimations'] = $zavataranimations;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-avataranimations.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: avatars.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple avatars information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$zuserid = $wtwconnect->userid;
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/avatars.php");

	/* get values from querystring or session */
	$zgroups = $wtwconnect->getVal('groups','');
	$zfilter = $wtwconnect->getVal('filter','mine');
	
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	$hasaccess = false;
	if ($zfilter == 'all') {
		$zroles = $wtwconnect->getUserRoles($zuserid);
		foreach ($zroles as $zrole) {
			if (strtolower($zrole['rolename']) == 'admin' || strtolower($zrole['rolename']) == 'architect' || strtolower($zrole['rolename']) == 'developer' || strtolower($zrole['rolename']) == 'graphics artist') {
				$hasaccess = true;
			}
		}
	}

	$zresults = array();
	$zwebtype = 'avatars';
	if ($wtwconnect->hasValue($zgroups)) {
		if ($zgroups == 'my') {
			/* pull a group of MY available avatars */
			$zresults = $wtwconnect->query("
				select distinct *,
					'' as hostuserid,
					'' as templatename, 
					'' as description, 
					'' as tags, 
					0 as sortorder, 
					'' as defaultdisplayname 
				from ".wtw_tableprefix."useravatars
				where userid='".$wtwconnect->userid."'
					and (not userid='')
					and deleted=0
				order by avatargroup, displayname, avatarid, useravatarid;");
			$zwebtype = 'useravatars';
		} else if ($zgroups == 'all') {
			/* pull all groups of available avatars */
			$zwhere = "a1.deleted=0 and (a1.hostuserid='".$zhostuserid."' or a1.hostuserid='') ";

			$zresults = $wtwconnect->query("
				select distinct a1.*,
					'' as useravatarid,
					u1.displayname as defaultdisplayname
				from ".wtw_tableprefix."avatars a1
				left join ".wtw_tableprefix."avatarsingroups ag1
                on a1.avatarid=ag1.avatarid
                left join ".wtw_tableprefix."avatargroups g1
                on g1.avatargroupid=ag1.avatargroupid
				left join (select displayname from ".wtw_tableprefix."users where userid='".$wtwconnect->userid."') u1
				on 1=1
				where ".$zwhere." 
				order by a1.hostuserid desc, a1.avatargroup, a1.displayname;");
		} else {
			/* pull a group of available avatars */
			$zwhere = "a1.deleted=0 and (a1.hostuserid='".$zhostuserid."' or a1.hostuserid='') and (";
			if (strpos($zgroups, ',') !== false) {
				$zgrouplist = explode(",", $zgroups);
				$i = 0;
				foreach ($zgrouplist as $zgroup) {
					if ($i == 0) {
						$zwhere .= "(a1.avatargroup='".$zgroup."' or g1.avatargroup='".$zgroup."') ";
					} else {
						$zwhere .= "or (a1.avatargroup='".$zgroup."' or g1.avatargroup='".$zgroup."') ";
					}
					$i += 1;
				}
				$zwhere .= ")";
			} else {
				$zwhere .= " (a1.avatargroup='".$zgroups."' or g1.avatargroup='".$zgroups."')) ";
			}

			$zresults = $wtwconnect->query("
				select distinct a1.*,
					'' as useravatarid,
					u1.displayname as defaultdisplayname
				from ".wtw_tableprefix."avatars a1
				left join ".wtw_tableprefix."avatarsingroups ag1
                on a1.avatarid=ag1.avatarid
                left join ".wtw_tableprefix."avatargroups g1
                on g1.avatargroupid=ag1.avatargroupid
				left join (select displayname from ".wtw_tableprefix."users where userid='".$wtwconnect->userid."') u1
				on 1=1
				where ".$zwhere." 
				order by a1.hostuserid desc, a1.avatargroup, a1.displayname;");
		}
	} else {
		if ($hasaccess) {
			$zresults = $wtwconnect->query("
				select distinct a1.*,
					'' as useravatarid,
					u1.displayname as defaultdisplayname
				from ".wtw_tableprefix."avatars a1
					left join (select displayname from ".wtw_tableprefix."users where userid='".$wtwconnect->userid."') u1
					on 1=1
				where a1.deleted=0
				order by a1.hostuserid desc, a1.avatargroup, a1.displayname;");
		} else {
			$zresults = $wtwconnect->query("
				select distinct a1.*,
					'' as useravatarid,
					u1.displayname as defaultdisplayname
				from ".wtw_tableprefix."avatars a1
					left join (select displayname from ".wtw_tableprefix."users where userid='".$wtwconnect->userid."') u1
					on 1=1
				where a1.deleted=0 and (a1.hostuserid='".$zhostuserid."' or a1.hostuserid='')
				order by a1.hostuserid desc, a1.avatargroup, a1.displayname;");
		}
	}
	$i = 0;
	$zavatars = array();

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	foreach ($zresults as $zrow) {
		$webid = $zrow["avatarid"];
		if ($zwebtype == 'useravatars') {
			$webid = $zrow["useravatarid"];
		}
		$zsnapshot = '';
		$zsnapshotthumbnail = '';
		
		if (file_exists(wtw_rootpath.'/content/uploads/'.$zwebtype.'/'.$webid.'/snapshots/defaultavatar.png')) {
			$zsnapshot = '/content/uploads/'.$zwebtype.'/'.$webid.'/snapshots/defaultavatar.png';
		} else {
			$zsnapshot = '/content/system/images/profilebig.png';
		}
		if (file_exists(wtw_rootpath.'/content/uploads/'.$zwebtype.'/'.$webid.'/snapshots/defaultavatarsm.png')) {
			$zsnapshotthumbnail = '/content/uploads/'.$zwebtype.'/'.$webid.'/snapshots/defaultavatarsm.png';
		} else {
			$zsnapshotthumbnail = '/content/system/images/menuprofilebig.png';
		}
	
		/* pull avatar groups by id */
		$zavatargroups = array();
		$zavatargroupsall = array();

		/* pull avatar groups by id */
		$zresults2 = $wtwconnect->query("
			select g1.*,
				ag1.avatarsingroupid
			from ".wtw_tableprefix."avatargroups g1 
				left join (select * from ".wtw_tableprefix."avatarsingroups where avatarid='".$zrow["avatarid"]."' and deleted=0) ag1
				on ag1.avatargroupid=g1.avatargroupid
			order by g1.avatargroup, ag1.avatarsingroupid;");
		$k = 0;
		$j = 0;
		foreach ($zresults2 as $zrow2) {
			$zavatarsingroupid = '';
			if (isset($zrow2["avatarsingroupid"]) && !empty($zrow2["avatarsingroupid"])) {
				$zavatarsingroupid = $zrow2["avatarsingroupid"];
			}
			$zavatargroupsall[$j] = array(
				'avatarsingroupid'=> $zavatarsingroupid,
				'avatargroupid'=> $zrow2["avatargroupid"],
				'avatargroup'=> $zrow2["avatargroup"]
			);
			$j += 1;
			if (!empty($zavatarsingroupid)) {
				$zavatargroups[$k] = array(
					'avatarsingroupid'=> $zrow2["avatarsingroupid"],
					'avatargroupid'=> $zrow2["avatargroupid"],
					'avatargroup'=> $zrow2["avatargroup"]
				);
				$k += 1;
			}
		}

		$zavatars[$i] = array(
			'useravatarid'=> $zrow["useravatarid"],
			'avatarid'=> $zrow["avatarid"],
			'hostuserid'=> $zrow["hostuserid"],
			'versionid'=> $zrow["versionid"],
			'version'=> $zrow["version"],
			'versionorder'=> $zrow["versionorder"],
			'versiondesc'=> $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'avatargroup'=> $zrow["avatargroup"],
			'avatargroups'=> $zavatargroups,
			'avatargroupsall'=> $zavatargroupsall,
			'displayname'=> $wtwconnect->escapeHTML($zrow["displayname"]),
			'defaultdisplayname'=> $zrow["defaultdisplayname"],
			'avatardescription'=> $wtwconnect->escapeHTML($zrow["avatardescription"]),
			'gender'=> $zrow["gender"],
			'objects'=> array(
				'folder'=> $zrow["objectfolder"],
				'file'=> $zrow["objectfile"],
				'startframe'=> $zrow["startframe"],
				'endframe'=> $zrow["endframe"]
			),
			'position'=> array(
				'x'=> $zrow["positionx"],
				'y'=> $zrow["positiony"],
				'z'=> $zrow["positionz"]
			),
			'scaling'=> array(
				'x'=> $zrow["scalingx"],
				'y'=> $zrow["scalingy"],
				'z'=> $zrow["scalingz"]
			),
			'rotation'=> array(
				'x'=> $zrow["rotationx"],
				'y'=> $zrow["rotationy"],
				'z'=> $zrow["rotationz"]
			),
			'snapshots'=> array(
				'full'=> $zsnapshot,
				'thumbnail'=> $zsnapshotthumbnail
			),
			'share'=> array(
				'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
				'description' => $wtwconnect->escapeHTML($zrow["description"]),
				'tags' => $wtwconnect->escapeHTML($zrow["tags"])
			),
			'sounds'=> array(
				'voice' => null
			),
			'sortorder'=> $zrow["sortorder"]
		);
		$i += 1;
	}
	$zresponse['avatars'] = $zavatars;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-avatars.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: building.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Building information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/building.php");
	
	/* get values from querystring or session */
	$zbuildingid = $wtwconnect->getVal('buildingid','');

	/* select building data */
	$zresults = $wtwconnect->query("
		select *,
			case when snapshotid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u1 
						where u1.uploadid=snapshotid limit 1)
				end as snapshotpath
		from ".wtw_tableprefix."buildings
		where buildingid='".$zbuildingid."'
		   and deleted=0;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array('userid'=> $zrow["userid"]);
		$zbuildinginfo = array(
			'buildingid' => $zrow["buildingid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'buildingname' => $wtwconnect->escapeHTML($zrow["buildingname"]),
			'buildingdescription' => $wtwconnect->escapeHTML($zrow["buildingdescription"]),
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["snapshotpath"],
			'analyticsid'=> $zrow["analyticsid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"]
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zresponse[$i] = array(
			'buildinginfo'=> $zbuildinginfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'alttag'=> $zalttag,
			'authorizedusers'=> $zauthorizedusers,
			'gravity'=> $zrow["gravity"]
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-building.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: buildingmoldsrecover.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides building mold information to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/buildingmoldsrecover.php");
	
	/* get values from querystring or session */
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zbuildingind = $wtwconnect->getVal('buildingind','-1');
	$zbuildingmoldid = $wtwconnect->getVal('buildingmoldid','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');

	/* select building mold for recovery */
	$zresults = $wtwconnect->query("
		select a1.*,
			buildings.analyticsid,
			case when a1.uploadobjectid = '' then ''
				else
					(select uo1.objectfolder 
						from ".wtw_tableprefix."uploadobjects uo1 
						where uo1.uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select uo1.objectfile 
						from ".wtw_tableprefix."uploadobjects uo1 
						where uo1.uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
			case when a1.textureid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
			case when a1.texturebumpid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
			case when a1.mixmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
			case when a1.texturerid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
			case when a1.texturegid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
			case when a1.texturebid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
			case when a1.texturebumprid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
			case when a1.texturebumpgid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
			case when a1.texturebumpbid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
			case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
			case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
			(select count(*) 
				from ".wtw_tableprefix."buildingmolds 
				where buildingid='".$zbuildingid."' 
					and csgmoldid=a1.buildingmoldid) as csgcount
		from (select * 
				from ".wtw_tableprefix."buildingmolds 
				where buildingid='".$zbuildingid."' 
					and buildingmoldid='".$zbuildingmoldid."' 
					and deleted=0) a1 
			left join (select * 
					from ".wtw_tableprefix."buildings 
					where buildingid='".$zbuildingid."' 
						and deleted=0) buildings
				on a1.buildingid = buildings.buildingid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$zmolds = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zobjectanimations = null;
		$ztempwebtext = "";
		if ($wtwconnect->hasValue($zrow["webtext"])) {
			$ztempwebtext = implode('',(array)$zrow["webtext"]);
		}
		if ($wtwconnect->hasValue($zrow["uploadobjectid"])) {
			$zobjectanimations = $wtwconnect->getobjectanimations($zrow["uploadobjectid"]);
		}
		$zcommunityinfo = array(
			'communityid'=> '',
			'communityind'=> '',
			'analyticsid'=> ''
		);
		$zbuildinginfo = array(
			'buildingid'=> $zrow["buildingid"],
			'buildingind'=> $zbuildingind,
			'analyticsid'=> ''
		);
		$zthinginfo = array(
			'thingid'=> '',
			'thingind'=> '',
			'analyticsid'=> ''
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"],
			'scroll'=>''
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"],
			'special1'=> $zrow["special1"],
			'special2'=> $zrow["special2"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"],
			'billboard'=> $zrow["billboard"]
		);
		$zcsg = array(
			'moldid'=> $zrow["csgmoldid"], 
			'moldind'=>'-1',
			'action'=> $zrow["csgaction"], 
			'count'=> $zrow["csgcount"]
		);
		$zobjects = array(
			'uploadobjectid'=> $zrow["uploadobjectid"], 
			'folder'=> $zrow["objectfolder"], 
			'file'=> $zrow["objectfile"],
			'objectanimations'=> $zobjectanimations,
			'light'=> '',
			'shadows'=> ''
		);
		$zgraphics = array(
			'texture'=> array(
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'bumpid'=> $zrow["texturebumpid"],
				'bumppath'=> $zrow["texturebumppath"],
				'videoid'=> $zrow["videoid"],
				'video'=> $zrow["video"],
				'videoposterid'=> $zrow["videoposterid"],
				'videoposter'=> $zrow["videoposter"],
				'backupid'=> ''
			),
			'heightmap'=> array(
				'original'=> '',
				'id'=> $zrow["heightmapid"],
				'path'=> $zrow["heightmappath"],
				'minheight'=> $zrow["minheight"],
				'maxheight'=> $zrow["maxheight"],
				'mixmapid'=> $zrow["mixmapid"],
				'mixmappath'=> $zrow["mixmappath"],
				'texturerid'=> $zrow["texturerid"],
				'texturerpath'=> $zrow["texturerpath"],
				'texturegid'=> $zrow["texturegid"],
				'texturegpath'=> $zrow["texturegpath"],
				'texturebid'=> $zrow["texturebid"],
				'texturebpath'=> $zrow["texturebpath"],
				'texturebumprid'=> $zrow["texturebumprid"],
				'texturebumprpath'=> $zrow["texturebumprpath"],
				'texturebumpgid'=> $zrow["texturebumpgid"],
				'texturebumpgpath'=> $zrow["texturebumpgpath"],
				'texturebumpbid'=> $zrow["texturebumpbid"],
				'texturebumpbpath'=> $zrow["texturebumpbpath"]
			),
			'uoffset'=> $zrow["uoffset"],
			'voffset'=> $zrow["voffset"],
			'uscale'=> $zrow["uscale"],
			'vscale'=> $zrow["vscale"],
			'level'=> $zrow["graphiclevel"],
			'receiveshadows'=> $zrow["receiveshadows"],
			'castshadows'=> $zrow["castshadows"],
			'waterreflection'=> $zrow["waterreflection"], 
			'webimages'=> $wtwconnect->getwebimages("", $zrow["buildingmoldid"], "",-1)
		);
		$zwebtext = array(
			'webtext'=> $zrow["webtext"],
			'fullheight'=> '0',
			'scrollpos'=> '0',
			'webstyle'=> $zrow["webstyle"]
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zpaths = array(
			'path1'=> $wtwconnect->getmoldpoints('', $zrow["buildingmoldid"], '', 1, $zrow["shape"]),
			'path2'=> $wtwconnect->getmoldpoints('', $zrow["buildingmoldid"], '', 2, $zrow["shape"])
		);
		$zcolor = array(
			'diffusecolor'=> $zrow["diffusecolor"],
			'emissivecolor'=> $zrow["emissivecolor"],
			'specularcolor'=> $zrow["specularcolor"],
			'ambientcolor'=> $zrow["ambientcolor"]
		);
		$zsound = array(
			'id' => $zrow["soundid"],
			'path' => $zrow["soundpath"],
			'name' => $zrow["soundname"],
			'attenuation' => $zrow["soundattenuation"],
			'loop' => $zrow["soundloop"],
			'maxdistance' => $zrow["soundmaxdistance"],
			'rollofffactor' => $zrow["soundrollofffactor"],
			'refdistance' => $zrow["soundrefdistance"],
			'coneinnerangle' => $zrow["soundconeinnerangle"],
			'coneouterangle' => $zrow["soundconeouterangle"],
			'coneoutergain' => $zrow["soundconeoutergain"],
			'sound' => ''
		);
		$zphysics = array(
			'enabled'=>$zrow["physicsenabled"],
			'center'=>array(
				'x'=>$zrow["physicscenterx"],
				'y'=>$zrow["physicscentery"],
				'z'=>$zrow["physicscenterz"]
			),
			'extents'=>array(
				'x'=>$zrow["physicsextentsx"],
				'y'=>$zrow["physicsextentsy"],
				'z'=>$zrow["physicsextentsz"]
			),
			'friction'=>$zrow["physicsfriction"],
			'istriggershape'=>$zrow["physicsistriggershape"],
			'mass'=>$zrow["physicsmass"],
			'pointa'=>array(
				'x'=>$zrow["physicspointax"],
				'y'=>$zrow["physicspointay"],
				'z'=>$zrow["physicspointaz"]
			),
			'pointb'=>array(
				'x'=>$zrow["physicspointbx"],
				'y'=>$zrow["physicspointby"],
				'z'=>$zrow["physicspointbz"]
			),
			'radius'=>$zrow["physicsradius"],
			'restitution'=>$zrow["physicsrestitution"],
			'rotation'=>array(
				'x'=>$zrow["physicsrotationx"],
				'y'=>$zrow["physicsrotationy"],
				'z'=>$zrow["physicsrotationz"],
				'w'=>$zrow["physicsrotationw"]
			),
			'startasleep'=>$zrow["physicsstartasleep"]
		);
		

				  `physicsenabled` int DEFAULT '0',
				  `physicscenterx` decimal(18,2) DEFAULT '0.00',
				  `physicscentery` decimal(18,2) DEFAULT '0.00',
				  `physicscenterz` decimal(18,2) DEFAULT '0.00',
				  `physicsextentsx` decimal(18,2) DEFAULT '0.00',
				  `physicsextentsy` decimal(18,2) DEFAULT '0.00',
				  `physicsextentsz` decimal(18,2) DEFAULT '0.00',
				  `physicsfriction` decimal(18,2) DEFAULT '0.00',
				  `physicsistriggershape` int DEFAULT '0',
				  `physicsmass` decimal(18,2) DEFAULT '0.00',
				  `physicspointax` decimal(18,2) DEFAULT '0.00',
				  `physicspointay` decimal(18,2) DEFAULT '0.00',
				  `physicspointaz` decimal(18,2) DEFAULT '0.00',
				  `physicspointbx` decimal(18,2) DEFAULT '0.00',
				  `physicspointby` decimal(18,2) DEFAULT '0.00',
				  `physicspointbz` decimal(18,2) DEFAULT '0.00',
				  `physicsradius` decimal(18,2) DEFAULT '0.00',
				  `physicsrestitution` decimal(18,2) DEFAULT '0.00',
				  `physicsrotationw` decimal(18,2) DEFAULT '0.00',
				  `physicsrotationx` decimal(18,2) DEFAULT '0.00',
				  `physicsrotationy` decimal(18,2) DEFAULT '0.00',
				  `physicsrotationz` decimal(18,2) DEFAULT '0.00',
				  `physicsstartasleep` int DEFAULT '0',


		
		$zmolds[$i] = array(
			'communityinfo'=> $zcommunityinfo, 
			'buildinginfo'=> $zbuildinginfo, 
			'thinginfo'=> $zthinginfo,
			'moldid'=> $zrow["buildingmoldid"],
			'moldind'=> '-1',
			'shape'=> $zrow["shape"], 
			'covering'=> $zrow["covering"], 
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'csg'=> $zcsg,
			'objects'=> $zobjects,
			'graphics'=> $zgraphics, 
			'webtext'=> $zwebtext, 
			'alttag'=> $zalttag,
			'paths'=> $zpaths,
			'color'=> $zcolor,
			'sound'=> $zsound,
			'physics'=> $zphysics,
			'subdivisions'=> $zrow["subdivisions"], 
			'subdivisionsshown'=>'0',
			'shown'=>'0',
			'opacity'=> $zrow["opacity"], 
			'checkcollisions'=> $zrow["checkcollisions"], 
			'ispickable'=> $zrow["ispickable"], 
			'jsfunction'=> '',
			'jsparameters'=> '',
			'actionzoneid'=> $zrow["actionzoneid"],
			'actionzone2id'=> $zrow["actionzone2id"],
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'unloadactionzoneid'=> $zrow["unloadactionzoneid"],
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'attachmoldind'=> '-1',
			'loaded'=> '0',
			'parentname'=>'',
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['molds'] = $zmolds;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-buildingmoldsrecover.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: buildingnames.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides a list of 3D Building names information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/buildingnames.php");
	
	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;

	/* select buildings by userid */
	$zresults = $wtwconnect->query("
		select b1.*
		from ".wtw_tableprefix."buildings b1
		where b1.deleted=0
		order by b1.buildingname, b1.buildingid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zbuildings = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zbuildings[$i] = array(
			'buildingid' => $zrow["buildingid"],
			'buildingname' => $wtwconnect->escapeHTML($zrow["buildingname"]),
			'buildingdescription' => $wtwconnect->escapeHTML($zrow["buildingdescription"])
		);
		$i += 1;
	}
	echo json_encode($zbuildings);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-buildingnames=".$e->getMessage());
}
?>

----------------------
----------------------
File: buildingrecoveritems.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides building mold information list to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/buildingrecoveritems.php");
	
	/* get values from querystring or session */
	$zbuildingid = $wtwconnect->getVal('buildingid','');

	/* select building molds that have been deleted */
	$zresults = $wtwconnect->query("
		select shape as item,
			buildingmoldid as itemid,
			'buildingmolds' as itemtype
		from ".wtw_tableprefix."buildingmolds
		where buildingid='".$zbuildingid."'
		   and deleted>0
		   and not deleteddate is null
		order by deleteddate desc, 
			buildingmoldid desc;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zresponse[$i] = array(
			'itemid'=> $zrow["itemid"], 
			'item'=> $zrow["item"],
			'itemtype'=> $zrow["itemtype"],
			'parentname'=>'');
		$i += 1;
	}
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-buildingrecoveritems.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: buildings.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Building information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/buildings.php");
	
	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;
	$zfilter = $wtwconnect->getVal('filter','mine');

	/* check user for global roles with access */
	$hasaccess = false;
	if ($zfilter == 'all') {
		$zroles = $wtwconnect->getUserRoles($zuserid);
		foreach ($zroles as $zrole) {
			if (strtolower($zrole['rolename']) == 'admin' || strtolower($zrole['rolename']) == 'architect' || strtolower($zrole['rolename']) == 'developer' || strtolower($zrole['rolename']) == 'graphics artist') {
				$hasaccess = true;
			}
		}
	}
	/* select buildings by userid */
	$zresults = array();
	if ($hasaccess) {
		/* user gas global access role that allows access */
		$zresults = $wtwconnect->query("
			select distinct b1.*,
				u1.filepath,
				u1.filetype,
				u1.filedata
			from (select * 
						from ".wtw_tableprefix."buildings
						where deleted=0) b1
				left join ".wtw_tableprefix."uploads u1
					on b1.snapshotid=u1.uploadid
			order by b1.buildingname, b1.buildingid;");
	} else {
		/* user will only receive data that they have granular permissions to view */
		$zresults = $wtwconnect->query("
			select distinct b1.*,
				u1.filepath,
				u1.filetype,
				u1.filedata
			from (select * 
					from ".wtw_tableprefix."userauthorizations 
					where userid='".$zuserid."' 
						and not buildingid='' 
						and deleted=0 
						and (useraccess='admin' 
							or useraccess='architect')) ua1
				inner join (select * 
						from ".wtw_tableprefix."buildings
						where deleted=0) b1
					on b1.buildingid = ua1.buildingid
				left join ".wtw_tableprefix."uploads u1
					on b1.snapshotid=u1.uploadid
			order by b1.buildingname, b1.buildingid;");
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array('userid'=> $zrow["userid"]);
		$snapshotdata = null;
		if ((!isset($zrow["filepath"]) || empty($zrow["filepath"])) && isset($zrow["filedata"]) && !empty($zrow["filedata"])) {
			$snapshotdata = "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["filedata"]));
		}
		$zbuildinginfo = array(
			'buildingid' => $zrow["buildingid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'buildingname' => $wtwconnect->escapeHTML($zrow["buildingname"]),
			'buildingdescription' => $wtwconnect->escapeHTML($zrow["buildingdescription"]),
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["filepath"],
			'analyticsid'=> $zrow["analyticsid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'snapshotdata'=> $snapshotdata
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zresponse[$i] = array(
			'buildinginfo'=> $zbuildinginfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'alttag'=> $zalttag,
			'authorizedusers'=> $zauthorizedusers,
			'gravity'=> $zrow["gravity"]
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-buildings=".$e->getMessage());
}
?>

----------------------
----------------------
File: communities.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple 3D Communities information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/communities.php");

	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;
	$zfilter = $wtwconnect->getVal('filter','mine');

	/* check user for global roles with access */
	$hasaccess = false;
	if ($zfilter == 'all') {
		$zroles = $wtwconnect->getUserRoles($zuserid);
		foreach ($zroles as $zrole) {
			if (strtolower($zrole['rolename']) == 'admin' || strtolower($zrole['rolename']) == 'architect' || strtolower($zrole['rolename']) == 'developer' || strtolower($zrole['rolename']) == 'graphics artist') {
				$hasaccess = true;
			}
		}
	}
	/* select communities by userid */
	$zresults = array();
	if ($hasaccess) {
		/* select communities based on global role */
		$zresults = $wtwconnect->query("
			select '".$wtwconnect->userid."' as useraccess,
				c1.communityid,
				c1.versionid,
				c1.version,
				c1.versionorder,
				c1.versiondesc,
				c1.communityname,
				c1.communitydescription,
				c1.snapshotid,
				c1.analyticsid,
				c1.gravity,
				c1.templatename,
				c1.description,
				c1.tags,
				c1.textureid,
				u2.filepath as texturepath,
				c1.skydomeid,
				c1.waterbumpid,
				u3.filepath as waterbumppath,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				c1.alttag,
				c1.buildingpositionx,
				c1.buildingpositiony,
				c1.buildingpositionz,
				c1.buildingscalingx,
				c1.buildingscalingy,
				c1.buildingscalingz,
				c1.buildingrotationx,
				c1.buildingrotationy,
				c1.buildingrotationz,
				c1.createdate,
				c1.createuserid,
				c1.updatedate,
				c1.updateuserid,
				u1.filepath,
				u1.filetype,
				az1.actionzoneid as extremeloadzoneid,
				max(u1.filedata) as filedata
			from ".wtw_tableprefix."communities c1
				left join ".wtw_tableprefix."uploads u1
					on c1.snapshotid=u1.uploadid
				left join ".wtw_tableprefix."uploads u2
					on c1.textureid=u2.uploadid
				left join ".wtw_tableprefix."uploads u3
					on c1.waterbumpid=u3.uploadid
				left join (select communityid, actionzoneid 
					from ".wtw_tableprefix."actionzones 
					where actionzonename like 'extreme%' 
						and not actionzonename like '%custom%' 
						and not communityid='') az1 
					on c1.communityid=az1.communityid
			where 
			   c1.deleted=0
			group by 
				useraccess,
				c1.communityid,
				c1.versionid,
				c1.version,
				c1.versionorder,
				c1.versiondesc,
				c1.communityname,
				c1.communitydescription,
				c1.snapshotid,
				c1.analyticsid,
				c1.gravity,
				c1.templatename,
				c1.description,
				c1.tags,
				c1.textureid,
				u2.filepath,
				c1.skydomeid,
				c1.waterbumpid,
				u3.filepath,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				c1.alttag,
				c1.buildingpositionx,
				c1.buildingpositiony,
				c1.buildingpositionz,
				c1.buildingscalingx,
				c1.buildingscalingy,
				c1.buildingscalingz,
				c1.buildingrotationx,
				c1.buildingrotationy,
				c1.buildingrotationz,
				c1.createdate,
				c1.createuserid,
				c1.updatedate,
				c1.updateuserid,
				u1.filepath,
				u1.filetype,
				az1.actionzoneid
			order by c1.communityname, 
				c1.communityid;");
	} else {
		/* select communities for user with granular permissions */
		$zresults = $wtwconnect->query("
			select ua1.useraccess,
				c1.communityid,
				c1.versionid,
				c1.version,
				c1.versionorder,
				c1.versiondesc,
				c1.communityname,
				c1.communitydescription,
				c1.snapshotid,
				c1.analyticsid,
				c1.gravity,
				c1.templatename,
				c1.description,
				c1.tags,
				c1.textureid,
				u2.filepath as texturepath,
				c1.skydomeid,
				c1.waterbumpid,
				u3.filepath as waterbumppath,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				c1.alttag,
				c1.buildingpositionx,
				c1.buildingpositiony,
				c1.buildingpositionz,
				c1.buildingscalingx,
				c1.buildingscalingy,
				c1.buildingscalingz,
				c1.buildingrotationx,
				c1.buildingrotationy,
				c1.buildingrotationz,
				c1.createdate,
				c1.createuserid,
				c1.updatedate,
				c1.updateuserid,
				u1.filepath,
				u1.filetype,
				az1.actionzoneid as extremeloadzoneid,
				max(u1.filedata) as filedata
			from ".wtw_tableprefix."userauthorizations ua1
				inner join ".wtw_tableprefix."communities c1
					on ua1.communityid = c1.communityid
				left join ".wtw_tableprefix."uploads u1
					on c1.snapshotid=u1.uploadid
				left join ".wtw_tableprefix."uploads u2
					on c1.textureid=u2.uploadid
				left join ".wtw_tableprefix."uploads u3
					on c1.waterbumpid=u3.uploadid
				left join (select communityid, actionzoneid 
					from ".wtw_tableprefix."actionzones 
					where actionzonename like 'extreme%' 
						and not actionzonename like '%custom%' 
						and not communityid='') az1 
					on c1.communityid=az1.communityid
			where ua1.userid='".$wtwconnect->userid."'
					and ua1.deleted=0
					and (ua1.useraccess='admin'
					or ua1.useraccess='architect')
			   and c1.deleted=0
			group by 
				ua1.useraccess,
				c1.communityid,
				c1.versionid,
				c1.version,
				c1.versionorder,
				c1.versiondesc,
				c1.communityname,
				c1.communitydescription,
				c1.snapshotid,
				c1.analyticsid,
				c1.gravity,
				c1.templatename,
				c1.description,
				c1.tags,
				c1.textureid,
				u2.filepath,
				c1.skydomeid,
				c1.waterbumpid,
				u3.filepath,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				c1.alttag,
				c1.buildingpositionx,
				c1.buildingpositiony,
				c1.buildingpositionz,
				c1.buildingscalingx,
				c1.buildingscalingy,
				c1.buildingscalingz,
				c1.buildingrotationx,
				c1.buildingrotationy,
				c1.buildingrotationz,
				c1.createdate,
				c1.createuserid,
				c1.updatedate,
				c1.updateuserid,
				u1.filepath,
				u1.filetype,
				az1.actionzoneid
			order by c1.communityname, 
				c1.communityid;");
	}

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array(
			'userid'=> $wtwconnect->userid
		);
		$snapshotdata = null;
		if ((!isset($zrow["filepath"]) || empty($zrow["filepath"])) && isset($zrow["filedata"]) && !empty($zrow["filedata"])) {
			$snapshotdata = "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["filedata"]));
		}
		$zcommunityinfo = array(
			'communityid' => $zrow["communityid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'communityname' => $wtwconnect->escapeHTML($zrow["communityname"]),
			'communitydescription' => $wtwconnect->escapeHTML($zrow["communitydescription"]),
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["filepath"],
			'analyticsid'=> $zrow["analyticsid"],
			'extremeloadzoneid' => $zrow["extremeloadzoneid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'snapshotdata'=> $snapshotdata
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zgraphics = array(
			'texture'=> array (
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'backupid'=>'',
				'backuppath'=>''
			),
			'sky'=> array (
				'id'=> $zrow["skydomeid"],
				'backupid'=>''						
			)
		);
		$zground = array(
			'position'=> array (
				'y'=> $zrow["groundpositiony"]
			)
		);
		$zwater = array(
			'bump'=> array (
				'id'=> $zrow["waterbumpid"],
				'path'=> $zrow["waterbumppath"],
				'height'=> $zrow["waterbumpheight"],
				'backupid'=>'',
				'backuppath'=>''
			),
			'position'=> array (
				'y'=> $zrow["waterpositiony"]
			),
			'subdivisions'=> $zrow["watersubdivisions"],
			'waveheight'=> $zrow["waterwaveheight"],
			'wavelength'=> $zrow["waterwavelength"],
			'colorrefraction'=> $zrow["watercolorrefraction"],
			'colorreflection'=> $zrow["watercolorreflection"],
			'colorblendfactor'=> $zrow["watercolorblendfactor"],
			'colorblendfactor2'=> $zrow["watercolorblendfactor2"],
			'alpha'=> $zrow["wateralpha"]
		);
		$zwind = array(
			'direction'=> array (
				'x'=> $zrow["winddirectionx"],
				'y'=> $zrow["winddirectiony"],
				'z'=> $zrow["winddirectionz"]
			),
			'force'=> $zrow["windforce"]
		);

		$zscene = array(
			'sceneambientcolor' => $zrow["sceneambientcolor"],
			'sceneclearcolor' => $zrow["sceneclearcolor"],
			'sceneuseclonedmeshmap' => $zrow["sceneuseclonedmeshmap"],
			'sceneblockmaterialdirtymechanism' => $zrow["sceneblockmaterialdirtymechanism"]
		);
		$zlight = array(
			'sundirectionalintensity' => $zrow["sundirectionalintensity"],
			'sundiffusecolor' => $zrow["sundiffusecolor"],
			'sunspecularcolor' => $zrow["sunspecularcolor"],
			'sungroundcolor' => $zrow["sungroundcolor"],
			'sundirectionx' => $zrow["sundirectionx"],
			'sundirectiony' => $zrow["sundirectiony"],
			'sundirectionz' => $zrow["sundirectionz"],
			'sunpositionx' => $zrow["sunpositionx"],
			'sunpositiony' => $zrow["sunpositiony"],
			'sunpositionz' => $zrow["sunpositionz"],
			'backlightintensity' => $zrow["backlightintensity"],
			'backlightdirectionx' => $zrow["backlightdirectionx"],
			'backlightdirectiony' => $zrow["backlightdirectiony"],
			'backlightdirectionz' => $zrow["backlightdirectionz"],
			'backlightpositionx' => $zrow["backlightpositionx"],
			'backlightpositiony' => $zrow["backlightpositiony"],
			'backlightpositionz' => $zrow["backlightpositionz"],
			'backlightdiffusecolor' => $zrow["backlightdiffusecolor"],
			'backlightspecularcolor' => $zrow["backlightspecularcolor"]
		);
		$zfog = array(
			'scenefogenabled' => $zrow["scenefogenabled"],
			'scenefogmode' => $zrow["scenefogmode"],
			'scenefogdensity' => $zrow["scenefogdensity"],
			'scenefogstart' => $zrow["scenefogstart"],
			'scenefogend' => $zrow["scenefogend"],
			'scenefogcolor' => $zrow["scenefogcolor"]
		);
		$zsky = array(
			'skytype' => $zrow["skytype"],
			'skysize' => $zrow["skysize"],
			'skyboxfolder' => $zrow["skyboxfolder"],
			'skyboxfile' => $zrow["skyboxfile"],
			'skyboximageleft' => $zrow["skyboximageleft"],
			'skyboximageup' => $zrow["skyboximageup"],
			'skyboximagefront'=> $zrow["skyboximagefront"],
			'skyboximageright' => $zrow["skyboximageright"],
			'skyboximagedown' => $zrow["skyboximagedown"],
			'skyboximageback' => $zrow["skyboximageback"],
			'skypositionoffsetx' => $zrow["skypositionoffsetx"],
			'skypositionoffsety' => $zrow["skypositionoffsety"],
			'skypositionoffsetz' => $zrow["skypositionoffsetz"],
			'skyboxmicrosurface' => $zrow["skyboxmicrosurface"],
			'skyboxpbr' => $zrow["skyboxpbr"],
			'skyboxasenvironmenttexture' => $zrow["skyboxasenvironmenttexture"],
			'skyboxblur' => $zrow["skyboxblur"],
			'skyboxdiffusecolor' => $zrow["skyboxdiffusecolor"],
			'skyboxspecularcolor' => $zrow["skyboxspecularcolor"],
			'skyboxambientcolor' => $zrow["skyboxambientcolor"],
			'skyboxemissivecolor' => $zrow["skyboxemissivecolor"],
			'skyinclination' => $zrow["skyinclination"],
			'skyluminance' => $zrow["skyluminance"],
			'skyazimuth' => $zrow["skyazimuth"],
			'skyrayleigh' => $zrow["skyrayleigh"],
			'skyturbidity' => $zrow["skyturbidity"],
			'skymiedirectionalg' => $zrow["skymiedirectionalg"],
			'skymiecoefficient' => $zrow["skymiecoefficient"]
		);

		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zfirstbuilding = array(
			'position' => array(
				'x'=> $zrow["buildingpositionx"], 
				'y'=> $zrow["buildingpositiony"], 
				'z'=> $zrow["buildingpositionz"]
			),
			'scaling' => array(
				'x'=> $zrow["buildingscalingx"], 
				'y'=> $zrow["buildingscalingy"], 
				'z'=> $zrow["buildingscalingz"]
			),
			'rotation' => array(
				'x'=> $zrow["buildingrotationx"], 
				'y'=> $zrow["buildingrotationy"], 
				'z'=> $zrow["buildingrotationz"]
			)
		);
		$zresponse[$i] = array(
			'communityinfo' => $zcommunityinfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'graphics' => $zgraphics,
			'ground' => $zground,
			'water' => $zwater,
			'wind' => $zwind,
			'scene' => $zscene,
			'light' => $zlight,
			'fog' => $zfog,
			'sky' => $zsky,
			'authorizedusers'=> $zauthorizedusers,
			'alttag'=> $zalttag,
			'firstbuilding'=> $zfirstbuilding,
			'gravity'=> $zrow["gravity"]
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-communities.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: community.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Community information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/community.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');

	/* select community data */
	$zresults = $wtwconnect->query("
		select c1.*,
			az1.actionzoneid as extremeloadzoneid,
			case when c1.textureid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.websizeid=u1.uploadid 
						where u2.uploadid=c1.textureid limit 1)
				end as texturepath,
			case when c1.textureid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.uploadid=u1.uploadid 
						where u2.uploadid=c1.textureid limit 1)
				end as texturepath2,
			case when c1.skydomeid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.websizeid=u1.uploadid 
						where u2.uploadid=c1.skydomeid limit 1)
				end as skydomepath,
			case when c1.skydomeid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.uploadid=u1.uploadid 
						where u2.uploadid=c1.skydomeid limit 1)
				end as skydomepath2,
			case when c1.waterbumpid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.websizeid=u1.uploadid 
						where u2.uploadid=c1.waterbumpid limit 1)
				end as waterbumppath,
			case when c1.waterbumpid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u2 
							left join ".wtw_tableprefix."uploads u1 
								on u2.uploadid=u1.uploadid 
						where u2.uploadid=c1.waterbumpid limit 1)
				end as waterbumppath2,
			case when c1.snapshotid = '' then ''
				else
					(select u1.filepath 
						from ".wtw_tableprefix."uploads u1 
						where u1.uploadid=c1.snapshotid limit 1)
				end as snapshotpath,
			case when (select GROUP_CONCAT(userid) as useraccess 
						from ".wtw_tableprefix."userauthorizations 
						where communityid='".$zcommunityid."' 
							and deleted=0 
							and not communityid='') is null then ''
				else
					(select GROUP_CONCAT(userid) as useraccess 
						from ".wtw_tableprefix."userauthorizations 
						where communityid='".$zcommunityid."' 
							and deleted=0 
							and not communityid='')
				end as communityaccess
		from ".wtw_tableprefix."communities c1 
			left join ".wtw_tableprefix."uploads u3
				on c1.textureid=u3.uploadid
			left join (select communityid, actionzoneid 
				from ".wtw_tableprefix."actionzones 
				where actionzonename like 'extreme%' 
					and not actionzonename like '%custom%' 
					and not communityid='') az1 
				on c1.communityid=az1.communityid
		where c1.communityid='".$zcommunityid."'
		   and c1.deleted=0;");

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$communities = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array(
			'userid'=> $zrow["userid"]
		);
		$zcommunityinfo = array(
			'communityid' => $zrow["communityid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'communityname' => $wtwconnect->escapeHTML($zrow["communityname"]),
			'communitydescription' => $wtwconnect->escapeHTML($zrow["communitydescription"]),
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["snapshotpath"],
			'analyticsid'=> $zrow["analyticsid"],
			'extremeloadzoneid' => $zrow["extremeloadzoneid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'access'=> $zrow["communityaccess"]
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zgraphics = array(
			'texture'=> array (
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'path2'=> $zrow["texturepath2"],
				'backupid'=>'',
				'backuppath'=>''
			),
			'sky'=> array (
				'id'=> $zrow["skydomeid"],
				'path'=> $zrow["skydomepath"],
				'path2'=> $zrow["skydomepath2"],
				'backupid'=>''
			)
		);
		$zground = array(
			'position'=> array (
				'y'=> $zrow["groundpositiony"]
			)
		);
		$zwater = array(
			'bump'=> array (
				'id'=> $zrow["waterbumpid"],
				'path'=> $zrow["waterbumppath"],
				'path2'=> $zrow["waterbumppath2"],
				'height'=> $zrow["waterbumpheight"],
				'backupid'=>'',
				'backuppath'=>''
			),
			'position'=> array (
				'y'=> $zrow["waterpositiony"]
			),
			'subdivisions'=> $zrow["watersubdivisions"],
			'waveheight'=> $zrow["waterwaveheight"],
			'wavelength'=> $zrow["waterwavelength"],
			'colorrefraction'=> $zrow["watercolorrefraction"],
			'colorreflection'=> $zrow["watercolorreflection"],
			'colorblendfactor'=> $zrow["watercolorblendfactor"],
			'colorblendfactor2'=> $zrow["watercolorblendfactor2"],
			'alpha'=> $zrow["wateralpha"]
		);
		$zwind = array(
			'direction'=> array (
				'x'=> $zrow["winddirectionx"],
				'y'=> $zrow["winddirectiony"],
				'z'=> $zrow["winddirectionz"]
			),
			'force'=> $zrow["windforce"]
		);
		$zscene = array(
			'sceneambientcolor' => $zrow["sceneambientcolor"],
			'sceneclearcolor' => $zrow["sceneclearcolor"],
			'sceneuseclonedmeshmap' => $zrow["sceneuseclonedmeshmap"],
			'sceneblockmaterialdirtymechanism' => $zrow["sceneblockmaterialdirtymechanism"]
		);
		$zlight = array(
			'sundirectionalintensity' => $zrow["sundirectionalintensity"],
			'sundiffusecolor' => $zrow["sundiffusecolor"],
			'sunspecularcolor' => $zrow["sunspecularcolor"],
			'sungroundcolor' => $zrow["sungroundcolor"],
			'sundirectionx' => $zrow["sundirectionx"],
			'sundirectiony' => $zrow["sundirectiony"],
			'sundirectionz' => $zrow["sundirectionz"],
			'sunpositionx' => $zrow["sunpositionx"],
			'sunpositiony' => $zrow["sunpositiony"],
			'sunpositionz' => $zrow["sunpositionz"],
			'backlightintensity' => $zrow["backlightintensity"],
			'backlightdirectionx' => $zrow["backlightdirectionx"],
			'backlightdirectiony' => $zrow["backlightdirectiony"],
			'backlightdirectionz' => $zrow["backlightdirectionz"],
			'backlightpositionx' => $zrow["backlightpositionx"],
			'backlightpositiony' => $zrow["backlightpositiony"],
			'backlightpositionz' => $zrow["backlightpositionz"],
			'backlightdiffusecolor' => $zrow["backlightdiffusecolor"],
			'backlightspecularcolor' => $zrow["backlightspecularcolor"]
		);
		$zfog = array(
			'scenefogenabled' => $zrow["scenefogenabled"],
			'scenefogmode' => $zrow["scenefogmode"],
			'scenefogdensity' => $zrow["scenefogdensity"],
			'scenefogstart' => $zrow["scenefogstart"],
			'scenefogend' => $zrow["scenefogend"],
			'scenefogcolor' => $zrow["scenefogcolor"]
		);
		$zsky = array(
			'skytype' => $zrow["skytype"],
			'skysize' => $zrow["skysize"],
			'skyboxfolder' => $zrow["skyboxfolder"],
			'skyboxfile' => $zrow["skyboxfile"],
			'skyboximageleft' => $zrow["skyboximageleft"],
			'skyboximageup' => $zrow["skyboximageup"],
			'skyboximagefront'=> $zrow["skyboximagefront"],
			'skyboximageright' => $zrow["skyboximageright"],
			'skyboximagedown' => $zrow["skyboximagedown"],
			'skyboximageback' => $zrow["skyboximageback"],
			'skypositionoffsetx' => $zrow["skypositionoffsetx"],
			'skypositionoffsety' => $zrow["skypositionoffsety"],
			'skypositionoffsetz' => $zrow["skypositionoffsetz"],
			'skyboxmicrosurface' => $zrow["skyboxmicrosurface"],
			'skyboxpbr' => $zrow["skyboxpbr"],
			'skyboxasenvironmenttexture' => $zrow["skyboxasenvironmenttexture"],
			'skyboxblur' => $zrow["skyboxblur"],
			'skyboxdiffusecolor' => $zrow["skyboxdiffusecolor"],
			'skyboxspecularcolor' => $zrow["skyboxspecularcolor"],
			'skyboxambientcolor' => $zrow["skyboxambientcolor"],
			'skyboxemissivecolor' => $zrow["skyboxemissivecolor"],
			'skyinclination' => $zrow["skyinclination"],
			'skyluminance' => $zrow["skyluminance"],
			'skyazimuth' => $zrow["skyazimuth"],
			'skyrayleigh' => $zrow["skyrayleigh"],
			'skyturbidity' => $zrow["skyturbidity"],
			'skymiedirectionalg' => $zrow["skymiedirectionalg"],
			'skymiecoefficient' => $zrow["skymiecoefficient"]
		);
		$zfirstbuilding = array(
			'position' => array(
				'x'=> $zrow["buildingpositionx"], 
				'y'=> $zrow["buildingpositiony"], 
				'z'=> $zrow["buildingpositionz"]
			),
			'scaling' => array(
				'x'=> $zrow["buildingscalingx"], 
				'y'=> $zrow["buildingscalingy"], 
				'z'=> $zrow["buildingscalingz"]
			),
			'rotation' => array(
				'x'=> $zrow["buildingrotationx"], 
				'y'=> $zrow["buildingrotationy"], 
				'z'=> $zrow["buildingrotationz"]
			)
		);
		$communities[$i] = array(
			'communityinfo' => $zcommunityinfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'firstbuilding'=> $zfirstbuilding,
			'graphics' => $zgraphics,
			'ground' => $zground,
			'water' => $zwater,
			'wind' => $zwind,
			'scene' => $zscene,
			'light' => $zlight,
			'fog' => $zfog,
			'sky' => $zsky,
			'authorizedusers'=> $zauthorizedusers,
			'gravity'=> $zrow["gravity"]
		);
		$i += 1;
	}
	$zresponse['communities'] = $communities;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-community.php=".$e->getMessage());
}
?>
----------------------
----------------------
File: communitymoldsrecover.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides community mold information to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/communitymoldsrecover.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zcommunityind = $wtwconnect->getVal('communityind','-1');
	$zcommunitymoldid = $wtwconnect->getVal('communitymoldid','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');

	/* select community mold for recovery */
	$zresults = $wtwconnect->query("
		select a1.*,
			c1.analyticsid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
			case when a1.textureid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
			case when a1.texturebumpid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
			case when a1.mixmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
			case when a1.texturerid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
			case when a1.texturegid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
			case when a1.texturebid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
			case when a1.texturebumprid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
			case when a1.texturebumpgid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
			case when a1.texturebumpbid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
			case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads where uploadid=a1.videoid limit 1)
				end as video,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select originalid 
							from ".wtw_tableprefix."uploads where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
							from ".wtw_tableprefix."uploads where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
			case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads where uploadid=a1.soundid limit 1)
				end as soundpath,
			(select count(*) 
				from ".wtw_tableprefix."communitymolds 
				where communityid='".$zcommunityid."' 
					and csgmoldid=a1.communitymoldid) as csgcount
		from ".wtw_tableprefix."communitymolds a1
			left join ".wtw_tableprefix."communities c1
				on a1.communityid =  c1.communityid
		where a1.communityid='".$zcommunityid."'
		   and a1.communitymoldid='".$zcommunitymoldid;."'");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$zmolds = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zobjectanimations = null;
		$ztempwebtext = "";
		if ($wtwconnect->hasValue($zrow["webtext"])) {
			$ztempwebtext = implode('',(array)$zrow["webtext"]);
		}
		if ($wtwconnect->hasValue($zrow["uploadobjectid"])) {
			$zobjectanimations = $wtwconnect->getobjectanimations($zrow["uploadobjectid"]);
		}
		$zcommunityinfo = array(
			'communityid'=> $zrow["communityid"],
			'communityind'=> $zcommunityind,
			'analyticsid'=> ''
		);
		$zbuildinginfo = array(
			'buildingid'=> '',
			'buildingind'=> '',
			'analyticsid'=> ''
		);
		$zthinginfo = array(
			'thingid'=> '',
			'thingind'=> '',
			'analyticsid'=> ''
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"],
			'scroll'=>''
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"],
			'special1'=> $zrow["special1"],
			'special2'=> $zrow["special2"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"],
			'billboard'=> $zrow["billboard"]
		);
		$zcsg = array(
			'moldid'=> $zrow["csgmoldid"], 
			'moldind'=>'-1',
			'action'=> $zrow["csgaction"], 
			'count'=> $zrow["csgcount"]
		);
		$zobjects = array(
			'uploadobjectid'=> $zrow["uploadobjectid"], 
			'folder'=> $zrow["objectfolder"], 
			'file'=> $zrow["objectfile"],
			'objectanimations'=> $zobjectanimations,
			'light'=> '',
			'shadows'=> ''
		);
		$zgraphics = array(
			'texture'=> array(
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'bumpid'=> $zrow["texturebumpid"],
				'bumppath'=> $zrow["texturebumppath"],
				'videoid'=> $zrow["videoid"],
				'video'=> $zrow["video"],
				'videoposterid'=> $zrow["videoposterid"],
				'videoposter'=> $zrow["videoposter"],
				'backupid'=> ''
			),
			'heightmap'=> array(
				'original'=> '',
				'id'=> $zrow["heightmapid"],
				'path'=> $zrow["heightmappath"],
				'minheight'=> $zrow["minheight"],
				'maxheight'=> $zrow["maxheight"],
				'mixmapid'=> $zrow["mixmapid"],
				'mixmappath'=> $zrow["mixmappath"],
				'texturerid'=> $zrow["texturerid"],
				'texturerpath'=> $zrow["texturerpath"],
				'texturegid'=> $zrow["texturegid"],
				'texturegpath'=> $zrow["texturegpath"],
				'texturebid'=> $zrow["texturebid"],
				'texturebpath'=> $zrow["texturebpath"],
				'texturebumprid'=> $zrow["texturebumprid"],
				'texturebumprpath'=> $zrow["texturebumprpath"],
				'texturebumpgid'=> $zrow["texturebumpgid"],
				'texturebumpgpath'=> $zrow["texturebumpgpath"],
				'texturebumpbid'=> $zrow["texturebumpbid"],
				'texturebumpbpath'=> $zrow["texturebumpbpath"]
			),
			'uoffset'=> $zrow["uoffset"],
			'voffset'=> $zrow["voffset"],
			'uscale'=> $zrow["uscale"],
			'vscale'=> $zrow["vscale"],
			'level'=> $zrow["graphiclevel"],
			'receiveshadows'=> $zrow["receiveshadows"],
			'castshadows'=> $zrow["castshadows"],
			'waterreflection'=> $zrow["waterreflection"], 
			'webimages'=> $wtwconnect->getwebimages("", "", $zrow["communitymoldid"],-1)
		);
		$zwebtext = array(
			'webtext'=> $zrow["webtext"],
			'fullheight'=> '0',
			'scrollpos'=> '0',
			'webstyle'=> $zrow["webstyle"]
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zpaths = array(
			'path1'=> $wtwconnect->getmoldpoints('', '', $zrow["communitymoldid"], 1, $zrow["shape"]),
			'path2'=> $wtwconnect->getmoldpoints('', '', $zrow["communitymoldid"], 2, $zrow["shape"])
		);
		$zcolor = array(
			'diffusecolor'=> $zrow["diffusecolor"],
			'emissivecolor'=> $zrow["emissivecolor"],
			'specularcolor'=> $zrow["specularcolor"],
			'ambientcolor'=> $zrow["ambientcolor"]
		);
		$zsound = array(
			'id' => $zrow["soundid"],
			'path' => $zrow["soundpath"],
			'name' => $zrow["soundname"],
			'attenuation' => $zrow["soundattenuation"],
			'loop' => $zrow["soundloop"],
			'maxdistance' => $zrow["soundmaxdistance"],
			'rollofffactor' => $zrow["soundrollofffactor"],
			'refdistance' => $zrow["soundrefdistance"],
			'coneinnerangle' => $zrow["soundconeinnerangle"],
			'coneouterangle' => $zrow["soundconeouterangle"],
			'coneoutergain' => $zrow["soundconeoutergain"],
			'sound' => ''
		);
		$zphysics = array(
			'enabled'=>$zrow["physicsenabled"],
			'center'=>array(
				'x'=>$zrow["physicscenterx"],
				'y'=>$zrow["physicscentery"],
				'z'=>$zrow["physicscenterz"]
			),
			'extents'=>array(
				'x'=>$zrow["physicsextentsx"],
				'y'=>$zrow["physicsextentsy"],
				'z'=>$zrow["physicsextentsz"]
			),
			'friction'=>$zrow["physicsfriction"],
			'istriggershape'=>$zrow["physicsistriggershape"],
			'mass'=>$zrow["physicsmass"],
			'pointa'=>array(
				'x'=>$zrow["physicspointax"],
				'y'=>$zrow["physicspointay"],
				'z'=>$zrow["physicspointaz"]
			),
			'pointb'=>array(
				'x'=>$zrow["physicspointbx"],
				'y'=>$zrow["physicspointby"],
				'z'=>$zrow["physicspointbz"]
			),
			'radius'=>$zrow["physicsradius"],
			'restitution'=>$zrow["physicsrestitution"],
			'rotation'=>array(
				'x'=>$zrow["physicsrotationx"],
				'y'=>$zrow["physicsrotationy"],
				'z'=>$zrow["physicsrotationz"],
				'w'=>$zrow["physicsrotationw"]
			),
			'startasleep'=>$zrow["physicsstartasleep"]
		);
		$zmolds[$i] = array(
			'communityinfo'=> $zcommunityinfo, 
			'buildinginfo'=> $zbuildinginfo, 
			'thinginfo'=> $zthinginfo, 
			'moldid'=> $zrow["communitymoldid"], 
			'moldind'=> '-1',
			'shape'=> $zrow["shape"], 
			'thingid'=> $zrow["thingid"],
			'thingparts'=> array(),
			'covering'=> $zrow["covering"], 
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'csg'=> $zcsg,
			'objects'=> $zobjects,
			'graphics'=> $zgraphics, 
			'webtext'=> $zwebtext, 
			'alttag'=> $zalttag,
			'paths'=> $zpaths,
			'color'=> $zcolor,
			'sound'=> $zsound,
			'physics'=> $zphysics,
			'subdivisions'=> $zrow["subdivisions"], 
			'subdivisionsshown'=>'0',
			'shown'=>'0',
			'opacity'=> $zrow["opacity"], 
			'checkcollisions'=> $zrow["checkcollisions"], 
			'ispickable'=> $zrow["ispickable"], 
			'jsfunction'=> '',
			'jsparameters'=> '',
			'actionzoneid'=> $zrow["actionzoneid"],
			'actionzone2id'=> $zrow["actionzone2id"],
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'unloadactionzoneid'=> $zrow["unloadactionzoneid"],
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'attachmoldind'=> '-1',
			'loaded'=> '0',
			'parentname'=>'',
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['molds'] = $zmolds;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-communitymoldsrecover.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: communitynames.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides a list of 3D Community Names information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/communitynames.php");

	/* select communities for user */
	$zresults = $wtwconnect->query("
		select c1.*
		from ".wtw_tableprefix."communities c1
		where c1.deleted=0
		order by c1.communityname, 
			c1.communityid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zcommunities = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zcommunities[$i] = array(
			'communityid' => $zrow["communityid"],
			'communityname' => $wtwconnect->escapeHTML($zrow["communityname"]),
			'communitydescription' => $wtwconnect->escapeHTML($zrow["communitydescription"])
		);
		$i += 1;
	}
	echo json_encode($zcommunities);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-communitynames.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: communityrecoveritems.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides community mold information list to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/communityrecoveritems.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');

	/* select community molds that have been deleted */
	$zresults = $wtwconnect->query("
		select shape as item,
			communitymoldid as itemid,
			'communitymolds' as itemtype
		from ".wtw_tableprefix."communitymolds
		where communityid='".$zcommunityid."'
		   and deleted>0
		   and not deleteddate is null
		order by deleteddate desc, 
			communitymoldid desc;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zresponse[$i] = array(
			'itemid'=> $zrow["itemid"], 
			'item'=> $zrow["item"],
			'itemtype'=> $zrow["itemtype"],
			'parentname'=>'');
		$i += 1;
	}
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-communityrecoveritems.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: connectinggrids.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides connecting grid information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
$zresponse = array(
	'webitems' => array()
);
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/connectinggrids.php");
	
	/* get values from querystring or session */
	$zparentwebid = $wtwconnect->getVal('parentwebid','');
	$zparentname = $wtwconnect->getVal('parentname','');
	$zuserid = $wtwconnect->userid;
	$zstartpositionx = $wtwconnect->getNumber('startpositionx',0);
	$zstartpositiony = $wtwconnect->getNumber('startpositiony',0);
	$zstartpositionz = $wtwconnect->getNumber('startpositionz',0);
	$zconnectinggridid = "";
	$zsubconnectinggridid = "";
	$zwebtype = "";
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	if ($wtwconnect->hasValue($zparentwebid)) {
		/* select connectinggridid for main childid for item */
		$zresults = $wtwconnect->query("
			select connectinggridid 
				from ".wtw_tableprefix."connectinggrids 
				where childwebid='".$zparentwebid."' and parentwebid='' and deleted=0 limit 1;");
		foreach ($zresults as $zrow) {
			$zconnectinggridid = $zrow["connectinggridid"];
		}
		if (!isset($zconnectinggridid) || empty($zconnectinggridid)) {
			/* select connectinggridid for item */
			$zresults = $wtwconnect->query("
				select connectinggridid 
					from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zparentwebid."' and deleted=0 limit 1;");
			foreach ($zresults as $zrow) {
				$zconnectinggridid = $zrow["connectinggridid"];
			}
			/* select webtype for for item */
			$zresults = $wtwconnect->query("
				select parentwebtype 
					from ".wtw_tableprefix."connectinggrids 
					where connectinggridid='".$zconnectinggridid."' limit 1;");
			foreach ($zresults as $zrow) {
				$zwebtype = $zrow["childwebtype"];
			}
		} else {
			/* select webtype for main childid for item */
			$zresults = $wtwconnect->query("
				select childwebtype 
					from ".wtw_tableprefix."connectinggrids 
					where connectinggridid='".$zconnectinggridid."' limit 1;");
			foreach ($zresults as $zrow) {
				$zwebtype = $zrow["childwebtype"];
			}
		}
		
		$zresults = array();
		if ($zwebtype = 'community')  {
			/* get connectinggrids for community */
			$zresults = $wtwconnect->query("
				select connectinggrids.connectinggridid,
					connectinggrids.parentserverfranchiseid,
					connectinggrids.parentwebid,
					connectinggrids.parentwebtype,
					connectinggrids.childserverfranchiseid,
					connectinggrids.childwebid,
					connectinggrids.childwebtype,
					connectinggrids.loadactionzoneid,
					connectinggrids.altloadactionzoneid,
					connectinggrids.unloadactionzoneid,
					connectinggrids.attachactionzoneid,
					connectinggrids.alttag,
					connectinggrids.positionx,
					connectinggrids.positiony,
					connectinggrids.positionz,
					connectinggrids.scalingx,
					connectinggrids.scalingy,
					connectinggrids.scalingz,
					connectinggrids.rotationx,
					connectinggrids.rotationy,
					connectinggrids.rotationz,
					round(sqrt(pow(connectinggrids.positionx-".$zstartpositionx.",2) + pow(connectinggrids.positiony-".$zstartpositiony.",2) + pow(connectinggrids.positionz-".$zstartpositionz.",2))) as distance,
					IF(connectinggrids.parentwebid IS NULL or connectinggrids.parentwebid = '', '1', '0') as isparent,
					parentcommunities.communityid as parentcommunityid,
					parentcommunities.communityname as parentcommunityname,
					parentcommunities.snapshotid as parentcommunitysnapshotid,
					case when parentcommunities.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=parentcommunities.snapshotid limit 1)
						end as parentcommunitysnapshoturl,
					parentcommunities.analyticsid as parentcommunityanalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where communityid=parentcommunities.communityid 
									and deleted=0 and not communityid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where communityid=parentcommunities.communityid 
									and deleted=0 and not communityid='')
						end as parentcommunityaccess,
					parentbuildings.buildingid as parentbuildingid,
					parentbuildings.buildingname as parentbuildingname,
					parentbuildings.snapshotid as parentbuildingsnapshotid,
					case when parentbuildings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=parentbuildings.snapshotid limit 1)
						end as parentbuildingsnapshoturl,
					parentbuildings.analyticsid as parentbuildinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=parentbuildings.buildingid 
									and deleted=0 and not buildingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=parentbuildings.buildingid 
									and deleted=0 and not buildingid='')
						end as parentbuildingaccess,
					parentthings.thingid as parentthingid,
					parentthings.thingname as parentthingname,
					parentthings.snapshotid as parentthingsnapshotid,
					case when parentthings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=parentthings.snapshotid limit 1)
						end as parentthingsnapshoturl,
					parentthings.analyticsid as parentthinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=parentthings.thingid 
									and deleted=0 and not thingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=parentthings.thingid 
									and deleted=0 and not thingid='')
						end as parentthingaccess,
					communities.communityid,
					communities.communityname,
					communities.snapshotid as communitysnapshotid,
					case when communities.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=communities.snapshotid limit 1)
						end as communitysnapshoturl,
					communities.analyticsid as communityanalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where communityid=communities.communityid 
									and deleted=0 and not communityid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where communityid=communities.communityid 
									and deleted=0 and not communityid='')
						end as communityaccess,
					buildings.buildingid,
					buildings.buildingname,
					buildings.snapshotid as buildingsnapshotid,
					case when buildings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=buildings.snapshotid limit 1)
						end as buildingsnapshoturl,
					buildings.analyticsid as buildinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=buildings.buildingid 
									and deleted=0 and not buildingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=buildings.buildingid 
									and deleted=0 and not buildingid='')
						end as buildingaccess,
					things.thingid,
					things.thingname,
					things.snapshotid as thingsnapshotid,
					case when things.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=things.snapshotid limit 1)
						end as thingsnapshoturl,
					things.analyticsid as thinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=things.thingid 
									and deleted=0 and not thingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=things.thingid 
									and deleted=0 and not thingid='')
						end as thingaccess,
					childconnectinggrids.connectinggridid as subconnectinggridid,
					childconnectinggrids.parentserverfranchiseid as subparentserverfranchiseid,
					childconnectinggrids.parentwebid as subparentwebid,
					childconnectinggrids.parentwebtype as subparentwebtype,
					childconnectinggrids.childserverfranchiseid as subchildserverfranchiseid,
					childconnectinggrids.childwebid as subchildwebid,
					childconnectinggrids.childwebtype as subchildwebtype,
					childconnectinggrids.loadactionzoneid as subloadactionzoneid,
					childconnectinggrids.altloadactionzoneid as subaltloadactionzoneid,
					childconnectinggrids.unloadactionzoneid as subunloadactionzoneid,
					childconnectinggrids.attachactionzoneid as subattachactionzoneid,
					childconnectinggrids.alttag as subalttag,
					childconnectinggrids.positionx as subpositionx,
					childconnectinggrids.positiony as subpositiony,
					childconnectinggrids.positionz as subpositionz,
					childconnectinggrids.scalingx as subscalingx,
					childconnectinggrids.scalingy as subscalingy,
					childconnectinggrids.scalingz as subscalingz,
					childconnectinggrids.rotationx as subrotationx,
					childconnectinggrids.rotationy as subrotationy,
					childconnectinggrids.rotationz as subrotationz,
					round(sqrt(pow(childconnectinggrids.positionx,2) + pow(childconnectinggrids.positiony,2) + pow(childconnectinggrids.positionz,2))) as subdistance,
					IF(childconnectinggrids.connectinggridid IS NULL or childconnectinggrids.connectinggridid = '', '0', '1') as issubchild,
					subchildthings.thingid as subchildthingid,
					subchildthings.thingname as subchildthingname,
					subchildthings.snapshotid as subchildthingsnapshotid,
					case when subchildthings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=subchildthings.snapshotid limit 1)
						end as subchildthingsnapshoturl,
					subchildthings.analyticsid as subchildthinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=subchildthings.thingid 
									and deleted=0 and not thingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=subchildthings.thingid 
									and deleted=0 and not thingid='')
						end as subchildthingaccess
				from 
					(select * from ".wtw_tableprefix."connectinggrids 
						where (parentwebid='".$zparentwebid."' 
						or (childwebid='".$zparentwebid."' and parentwebid=''))  
						and deleted=0) connectinggrids 
					
					left join (select * from ".wtw_tableprefix."communities 
							where communityid='".$zparentwebid."' 
								and deleted=0) parentcommunities
						on ((parentcommunities.communityid = connectinggrids.parentwebid
						and connectinggrids.parentwebtype='community')
						or (parentcommunities.communityid = connectinggrids.childwebid 
						and connectinggrids.parentwebid=''
						and connectinggrids.childwebtype='community'))
					
					left join (select * from ".wtw_tableprefix."buildings 
							where buildingid='".$zparentwebid."' 
								and deleted=0) parentbuildings
						on ((connectinggrids.parentwebid = parentbuildings.buildingid
						and connectinggrids.parentwebtype='building')
						or (parentbuildings.buildingid = connectinggrids.childwebid 
						and connectinggrids.parentwebid=''
						and connectinggrids.childwebtype='building'))

					left join (select * from ".wtw_tableprefix."things 
							where thingid='".$zparentwebid."' and deleted=0) parentthings
						on ((connectinggrids.parentwebid = parentthings.thingid
						and connectinggrids.parentwebtype='thing')
						or (parentbuildings.buildingid = connectinggrids.childwebid 
						and connectinggrids.parentwebid=''
						and connectinggrids.childwebtype='thing'))

					left join (select * from ".wtw_tableprefix."communities 
							where deleted=0) communities
						on connectinggrids.childwebid = communities.communityid
						and connectinggrids.childwebtype='community'

					left join (select * from ".wtw_tableprefix."buildings 
							where deleted=0) buildings
						on connectinggrids.childwebid = buildings.buildingid
						and connectinggrids.childwebtype='building'

					left join (select * from ".wtw_tableprefix."things 
							where deleted=0) things
						on connectinggrids.childwebid = things.thingid
						and connectinggrids.childwebtype='thing'
				
					left join (select * from ".wtw_tableprefix."connectinggrids 
							where childwebtype='thing' and deleted=0) childconnectinggrids
						on connectinggrids.childwebid = childconnectinggrids.parentwebid
						and connectinggrids.childwebtype='building'

					left join (select * from ".wtw_tableprefix."things 
							where deleted=0) subchildthings
						on childconnectinggrids.childwebid = subchildthings.thingid
						and childconnectinggrids.childwebtype='thing'
				
				order by isparent desc,
					issubchild,
					distance,
					subdistance,
					connectinggrids.connectinggridid,
					connectinggrids.childwebid,
					connectinggrids.positionx,
					connectinggrids.positiony,
					connectinggrids.positionz;");
		} else {
			/* select connectinggrids for building or thing */
			$zresults = $wtwconnect->query("
				select connectinggrids.connectinggridid,
					connectinggrids.parentserverfranchiseid,
					connectinggrids.parentwebid,
					connectinggrids.parentwebtype,
					connectinggrids.childserverfranchiseid,
					connectinggrids.childwebid,
					connectinggrids.childwebtype,
					connectinggrids.loadactionzoneid,
					connectinggrids.altloadactionzoneid,
					connectinggrids.unloadactionzoneid,
					connectinggrids.attachactionzoneid,
					connectinggrids.alttag,
					connectinggrids.positionx,
					connectinggrids.positiony,
					connectinggrids.positionz,
					connectinggrids.scalingx,
					connectinggrids.scalingy,
					connectinggrids.scalingz,
					connectinggrids.rotationx,
					connectinggrids.rotationy,
					connectinggrids.rotationz,
					round(sqrt(pow(connectinggrids.positionx-".$zstartpositionx.",2) + pow(connectinggrids.positiony-".$zstartpositiony.",2) + pow(connectinggrids.positionz-".$zstartpositionz.",2))) as distance,
					IF(connectinggrids.parentwebid IS NULL or connectinggrids.parentwebid = '', '1', '0') as isparent,
					'' as parentcommunityid,
					'' as parentcommunityname,
					'' as parentcommunitysnapshotid,
					'' as parentcommunitysnapshoturl,
					'' as parentcommunityanalyticsid,
					'' as parentcommunityaccess,
					parentbuildings.buildingid as parentbuildingid,
					parentbuildings.buildingname as parentbuildingname,
					parentbuildings.snapshotid as parentbuildingsnapshotid,
					case when parentbuildings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=parentbuildings.snapshotid limit 1)
						end as parentbuildingsnapshoturl,
					parentbuildings.analyticsid as parentbuildinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=parentbuildings.buildingid 
									and deleted=0 and not buildingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=parentbuildings.buildingid 
									and deleted=0 and not buildingid='')
						end as parentbuildingaccess,
					parentthings.thingid as parentthingid,
					parentthings.thingname as parentthingname,
					parentthings.snapshotid as parentthingsnapshotid,
					case when parentthings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=parentthings.snapshotid limit 1)
						end as parentthingsnapshoturl,
					parentthings.analyticsid as parentthinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=parentthings.thingid 
									and deleted=0 and not thingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=parentthings.thingid 
									and deleted=0 and not thingid='')
						end as parentthingaccess,
					'' as communityid,
					'WalkTheWeb' as communityname,
					'' as communitysnapshotid,
					'' as communitysnapshoturl,
					'' as communityanalyticsid,
					'' as communityaccess,
					buildings.buildingid,
					buildings.buildingname,
					buildings.snapshotid as buildingsnapshotid,
					case when buildings.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=buildings.snapshotid limit 1)
						end as buildingsnapshoturl,
					buildings.analyticsid as buildinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=buildings.buildingid 
									and deleted=0 and not buildingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where buildingid=buildings.buildingid 
									and deleted=0 and not buildingid='')
						end as buildingaccess,
					things.thingid,
					things.thingname,
					things.snapshotid as thingsnapshotid,
					case when things.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=things.snapshotid limit 1)
						end as thingsnapshoturl,
					things.analyticsid as thinganalyticsid,
					case when (select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=things.thingid 
									and deleted=0 and not thingid='') is null then ''
						else
							(select GROUP_CONCAT(userid) as useraccess 
								from ".wtw_tableprefix."userauthorizations 
								where thingid=things.thingid 
									and deleted=0 and not thingid='')
						end as thingaccess,
					null as subconnectinggridid,
					'' as subparentserverfranchiseid,
					'' as subparentwebid,
					'' as subparentwebtype,
					'' as subchildserverfranchiseid,
					'' as subchildwebid,
					'' as subchildwebtype,
					'' as subloadactionzoneid,
					'' as subaltloadactionzoneid,
					'' as subunloadactionzoneid,
					'' as subattachactionzoneid,
					'' as subalttag,
					'0' as subpositionx,
					'0' as subpositiony,
					'0' as subpositionz,
					'1' as subscalingx,
					'1' as subscalingy,
					'1' as subscalingz,
					'0' as subrotationx,
					'0' as subrotationy,
					'0' as subrotationz,
					'0' as subdistance,
					'0' as issubchild,
					'' as subchildthingid,
					'' as subchildthingname,
					'' as subchildthingsnapshotid,
					'' as subchildthingsnapshoturl,
					'' as subchildthinganalyticsid,
					'' as subchildthingaccess
				from 
					(select * from ".wtw_tableprefix."connectinggrids 
						where (parentwebid='".$zparentwebid."' 
							or (childwebid='".$zparentwebid."' 
								and parentwebid=''))  
							and deleted=0) connectinggrids 
					
					left join (select * 
							from ".wtw_tableprefix."buildings 
							where buildingid='".$zparentwebid."' 
								and deleted=0) parentbuildings
						on ((connectinggrids.parentwebid = parentbuildings.buildingid
						and connectinggrids.parentwebtype='building')
						or (parentbuildings.buildingid = connectinggrids.childwebid 
						and connectinggrids.parentwebid=''
						and connectinggrids.childwebtype='building'))

					left join (select * 
							from ".wtw_tableprefix."things 
							where thingid='".$zparentwebid."' 
								and deleted=0 and deleted>0) parentthings
						on ((connectinggrids.parentwebid = parentthings.thingid
						and connectinggrids.parentwebtype='thing')
						or (parentbuildings.buildingid = connectinggrids.childwebid 
						and connectinggrids.parentwebid=''
						and connectinggrids.childwebtype='thing'))

					left join (select * from ".wtw_tableprefix."buildings 
							where deleted=0) buildings
						on connectinggrids.childwebid = buildings.buildingid
						and connectinggrids.childwebtype='building'

					left join (select * from ".wtw_tableprefix."things 
							where deleted=0) things
						on connectinggrids.childwebid = things.thingid
						and connectinggrids.childwebtype='thing'
				
					left join (select * from ".wtw_tableprefix."connectinggrids 
							where childwebtype='thing' 
								and deleted=0 and deleted>0) childconnectinggrids
						on connectinggrids.childwebid = childconnectinggrids.parentwebid
						and connectinggrids.childwebtype='building'

					left join (select * from ".wtw_tableprefix."things 
							where deleted=0 and deleted>0) subchildthings
						on childconnectinggrids.childwebid = subchildthings.thingid
						and childconnectinggrids.childwebtype='thing'
				
				order by isparent desc,
					issubchild,
					distance,
					subdistance,
					connectinggrids.connectinggridid,
					connectinggrids.childwebid,
					connectinggrids.positionx,
					connectinggrids.positiony,
					connectinggrids.positionz;");
		}

		$i = 0;
		$zconnectinggridid = "";
		$zsubconnectinggridid = "";
		$zwebitems = array();
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			if ($zconnectinggridid != $zrow["connectinggridid"] && $wtwconnect->hasValue($zrow["connectinggridid"])) {
				if ($wtwconnect->hasValue($zrow["parentcommunityid"])) {
					$zcommunityinfo = array(
						'communityid'=> $zrow["parentcommunityid"],
						'communityname'=> $wtwconnect->escapeHTML($zrow["parentcommunityname"]),
						'snapshotid' => $zrow["parentcommunitysnapshotid"],
						'snapshoturl' => $zrow["parentcommunitysnapshoturl"],
						'analyticsid'=> $zrow["parentcommunityanalyticsid"],
						'access'=> $zrow["parentcommunityaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["communityid"])) {
					$zcommunityinfo = array(
						'communityid'=> $zrow["communityid"],
						'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
						'snapshotid' => $zrow["communitysnapshotid"],
						'snapshoturl' => $zrow["communitysnapshoturl"],
						'analyticsid'=> $zrow["communityanalyticsid"],
						'access'=> $zrow["communityaccess"]
					);
				} else {
					$zcommunityinfo = array(
						'communityid'=> '',
						'communityname'=> 'WalkTheWeb',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				if ($wtwconnect->hasValue($zrow["parentbuildingid"])) {
					$zbuildinginfo = array(
						'buildingid'=> $zrow["parentbuildingid"], 
						'buildingname'=> $zrow["parentbuildingname"],
						'snapshotid' => $zrow["parentbuildingsnapshotid"],
						'snapshoturl' => $zrow["parentbuildingsnapshoturl"],
						'analyticsid'=> $zrow["parentbuildinganalyticsid"],
						'access'=> $zrow["parentbuildingaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["buildingid"])) {
					$zbuildinginfo = array(
						'buildingid'=> $zrow["buildingid"], 
						'buildingname'=> $zrow["buildingname"],
						'snapshotid' => $zrow["buildingsnapshotid"],
						'snapshoturl' => $zrow["buildingsnapshoturl"],
						'analyticsid'=> $zrow["buildinganalyticsid"],
						'access'=> $zrow["buildingaccess"]
					);
				} else {
					$zbuildinginfo = array(
						'buildingid'=> '', 
						'buildingname'=> '',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				if ($wtwconnect->hasValue($zrow["parentthingid"])) {
					$zthinginfo = array(
						'thingid'=> $zrow["parentthingid"], 
						'thingname'=> $zrow["parentthingname"],
						'snapshotid' => $zrow["parentthingsnapshotid"],
						'snapshoturl' => $zrow["parentthingsnapshoturl"],
						'analyticsid'=> $zrow["parentthinganalyticsid"],
						'access'=> $zrow["parentthingaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["thingid"])) {
					$zthinginfo = array(
						'thingid'=> $zrow["thingid"], 
						'thingname'=> $zrow["thingname"],
						'snapshotid' => $zrow["thingsnapshotid"],
						'snapshoturl' => $zrow["thingsnapshoturl"],
						'analyticsid'=> $zrow["thinganalyticsid"],
						'access'=> $zrow["thingaccess"]
					);
				} else {
					$zthinginfo = array(
						'thingid'=> '', 
						'thingname'=> '',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				$zposition = array(
					'x'=> $zrow["positionx"], 
					'y'=> $zrow["positiony"], 
					'z'=> $zrow["positionz"]
				);
				$zscaling = array(
					'x'=> $zrow["scalingx"], 
					'y'=> $zrow["scalingy"], 
					'z'=> $zrow["scalingz"]
				);
				$zrotation = array(
					'x'=> $zrow["rotationx"], 
					'y'=> $zrow["rotationy"], 
					'z'=> $zrow["rotationz"]
				);
				$zalttag = array(
					'name'=> $zrow['alttag']
				);
				$zwebitems[$i] = array(
					'serverfranchiseid' => '',
					'connectinggridid'=> $zrow["connectinggridid"], 
					'connectinggridind'=> '-1',
					'parentconnectinggridid'=> '', 
					'parentconnectinggridind'=> '-1',
					'loadlevel'=> '1',
					'parentserverfranchiseid'=> $zrow["parentserverfranchiseid"], 
					'parentwebid'=> $zrow["parentwebid"], 
					'parentwebtype'=> $zrow["parentwebtype"], 
					'childserverfranchiseid'=> $zrow["childserverfranchiseid"], 
					'childwebid'=> $zrow["childwebid"], 
					'childwebtype'=> $zrow["childwebtype"], 
					'loadactionzoneid'=> $zrow["loadactionzoneid"], 
					'loadactionzoneind'=> '-1', 
					'altloadactionzoneid'=> $zrow["altloadactionzoneid"], 
					'altloadactionzoneind'=> '-1', 
					'unloadactionzoneid'=> $zrow["unloadactionzoneid"], 
					'unloadactionzoneind'=> '-1', 
					'attachactionzoneid'=> $zrow["attachactionzoneid"], 
					'attachactionzoneind'=> '-1', 
					'communityinfo'=> $zcommunityinfo,
					'buildinginfo'=> $zbuildinginfo,
					'thinginfo'=> $zthinginfo,
					'position'=> $zposition,
					'scaling'=> $zscaling,
					'rotation'=> $zrotation,
					'alttag'=> $zalttag,
					'shape'=>'box',
					'ispickable'=>'0',
					'checkcollisions'=>'0',
					'moldname'=>'',
					'parentname'=>$zparentname,
					'shown'=>'0',
					'status'=> '0');
				$i += 1;
			}
			if ($zrow["parentwebtype"] == 'community' && $zsubconnectinggridid != $zrow["subconnectinggridid"] && isset($zrow["subconnectinggridid"]) && !empty($zrow["subconnectinggridid"])) {
				if ($wtwconnect->hasValue($zrow["parentcommunityid"])) {
					$zcommunityinfo = array(
						'communityid'=> $zrow["parentcommunityid"],
						'communityname'=> $wtwconnect->escapeHTML($zrow["parentcommunityname"]),
						'snapshotid' => $zrow["parentcommunitysnapshotid"],
						'snapshoturl' => $zrow["parentcommunitysnapshoturl"],
						'analyticsid'=> $zrow["parentcommunityanalyticsid"],
						'access'=> $zrow["parentcommunityaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["communityid"])) {
					$zcommunityinfo = array(
						'communityid'=> $zrow["communityid"],
						'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
						'snapshotid' => $zrow["communitysnapshotid"],
						'snapshoturl' => $zrow["communitysnapshoturl"],
						'analyticsid'=> $zrow["communityanalyticsid"],
						'access'=> $zrow["communityaccess"]
					);
				} else {
					$zcommunityinfo = array(
						'communityid'=> '',
						'communityname'=> 'WalkTheWeb',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				if ($wtwconnect->hasValue($zrow["parentbuildingid"])) {
					$zbuildinginfo = array(
						'buildingid'=> $zrow["parentbuildingid"], 
						'buildingname'=> $zrow["parentbuildingname"],
						'snapshotid' => $zrow["parentbuildingsnapshotid"],
						'snapshoturl' => $zrow["parentbuildingsnapshoturl"],
						'analyticsid'=> $zrow["parentbuildinganalyticsid"],
						'access'=> $zrow["parentbuildingaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["buildingid"])) {
					$zbuildinginfo = array(
						'buildingid'=> $zrow["buildingid"], 
						'buildingname'=> $zrow["buildingname"],
						'snapshotid' => $zrow["buildingsnapshotid"],
						'snapshoturl' => $zrow["buildingsnapshoturl"],
						'analyticsid'=> $zrow["buildinganalyticsid"],
						'access'=> $zrow["buildingaccess"]
					);
				} else {
					$zbuildinginfo = array(
						'buildingid'=> '', 
						'buildingname'=> '',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				if ($wtwconnect->hasValue($zrow["subchildthingid"])) {
					$zthinginfo = array(
						'thingid'=> $zrow["subchildthingid"], 
						'thingname'=> $zrow["subchildthingname"],
						'snapshotid' => $zrow["subchildthingsnapshotid"],
						'snapshoturl' => $zrow["subchildthingsnapshoturl"],
						'analyticsid'=> $zrow["subchildthinganalyticsid"],
						'access'=> $zrow["subchildthingaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["parentthingid"])) {
					$zthinginfo = array(
						'thingid'=> $zrow["parentthingid"], 
						'thingname'=> $zrow["parentthingname"],
						'snapshotid' => $zrow["parentthingsnapshotid"],
						'snapshoturl' => $zrow["parentthingsnapshoturl"],
						'analyticsid'=> $zrow["parentthinganalyticsid"],
						'access'=> $zrow["parentthingaccess"]
					);
				} elseif ($wtwconnect->hasValue($zrow["thingid"])) {
					$zthinginfo = array(
						'thingid'=> $zrow["thingid"], 
						'thingname'=> $zrow["thingname"],
						'snapshotid' => $zrow["thingsnapshotid"],
						'snapshoturl' => $zrow["thingsnapshoturl"],
						'analyticsid'=> $zrow["thinganalyticsid"],
						'access'=> $zrow["thingaccess"]
					);
				} else {
					$zthinginfo = array(
						'thingid'=> '', 
						'thingname'=> '',
						'snapshotid' => '',
						'snapshoturl' => '',
						'analyticsid'=> '',
						'access'=> ''
					);
				}
				$zalttag = array(
					'name'=> $zrow['subalttag']
				);
				$zposition = array(
					'x'=> $zrow["subpositionx"], 
					'y'=> $zrow["subpositiony"], 
					'z'=> $zrow["subpositionz"]
				);
				$zscaling = array(
					'x'=> $zrow["subscalingx"], 
					'y'=> $zrow["subscalingy"], 
					'z'=> $zrow["subscalingz"]
				);
				$zrotation = array(
					'x'=> $zrow["subrotationx"], 
					'y'=> $zrow["subrotationy"], 
					'z'=> $zrow["subrotationz"]
				);
				$zwebitems[$i] = array(
					'serverfranchiseid' => '',
					'connectinggridid'=> $zrow["subconnectinggridid"], 
					'connectinggridind'=> '-1',
					'parentconnectinggridid'=> $zrow["connectinggridid"], 
					'parentconnectinggridind'=> '-1',
					'loadlevel'=> '2',
					'parentserverfranchiseid'=> $zrow["subparentserverfranchiseid"], 
					'parentwebid'=> $zrow["subparentwebid"], 
					'parentwebtype'=> $zrow["subparentwebtype"], 
					'childserverfranchiseid'=> $zrow["subchildserverfranchiseid"], 
					'childwebid'=> $zrow["subchildwebid"], 
					'childwebtype'=> $zrow["subchildwebtype"], 
					'loadactionzoneid'=> $zrow["subloadactionzoneid"], 
					'loadactionzoneind'=> '-1', 
					'altloadactionzoneid'=> $zrow["subaltloadactionzoneid"], 
					'altloadactionzoneind'=> '-1', 
					'unloadactionzoneid'=> $zrow["subunloadactionzoneid"], 
					'unloadactionzoneind'=> '-1', 
					'attachactionzoneid'=> $zrow["subattachactionzoneid"], 
					'attachactionzoneind'=> '-1', 
					'communityinfo'=> $zcommunityinfo,
					'buildinginfo'=> $zbuildinginfo,
					'thinginfo'=> $zthinginfo,
					'position'=> $zposition,
					'scaling'=> $zscaling,
					'rotation'=> $zrotation,
					'alttag'=> $zalttag,
					'shape'=>'box',
					'ispickable'=>'0',
					'checkcollisions'=>'0',
					'moldname'=>'',
					'parentname'=>'',
					'shown'=>'0',
					'status'=> '0');
				$i += 1;
				$zsubconnectinggridid = $zrow["subconnectinggridid"];
			}
			$zconnectinggridid = $zrow["connectinggridid"];
		}
		$zresponse['webitems'] = $zwebitems;
	}
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-connectinggrids.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: dashboard.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic Admin Dashboard information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/dashboard.php");
	
	/* get values from querystring or session */

	$zwebsitesize = 0;
    $zdownloads = array();
    $zresponse = array();
	if ($wtwconnect->isUserInRole('admin')) {
		
		$zwebsitesize = $wtwconnect->dirSize($wtwconnect->rootpath);
		
		/* check download queue for any pending */
		$i = 0;
		$zresults = $wtwconnect->query("
			select * 
				from ".wtw_tableprefix."downloads 
				where deleted=0 
				order by createdate desc, downloadid desc;");
		foreach ($zresults as $zrow) {
			$zdownloads[$i] = array(
				"downloadid" => $zrow["downloadid"],
				"webid" => $zrow["webid"],
				"webtype" => $zrow["webtype"],
				"userip" => $zrow["userip"],
				"fromurl" => $zrow["fromurl"],
				"createdate" => $zrow["createdate"]
			);
			$i += 1;
		}
		
		/* get server totals */
		$i = 0;
		$zresults = $wtwconnect->query("
			select '3D Communities' as item, c1.scount, c2.mycount 
				from (select count(communityid) as scount from ".wtw_tableprefix."communities where deleted=0) c1 
					left join (select count(communityid) as mycount 
						from ".wtw_tableprefix."communities 
						where createuserid='".$wtwconnect->userid."' and deleted=0) c2 on 1=1
				union
			select '3D Buildings' as item, b1.scount, b2.mycount 
				from (select count(buildingid) as scount from ".wtw_tableprefix."buildings where deleted=0) b1
					left join (select count(buildingid) as mycount 
						from ".wtw_tableprefix."buildings 
						where createuserid='".$wtwconnect->userid."' and deleted=0) b2 on 1=1
				union
			select '3D Things' as item, t1.scount, t2.mycount 
				from (select count(thingid) as scount from ".wtw_tableprefix."things where deleted=0) t1
					left join (select count(thingid) as mycount 
						from ".wtw_tableprefix."things 
						where createuserid='".$wtwconnect->userid."' and deleted=0) t2 on 1=1
				union
			select '3D Avatars' as item, a1.scount, a2.mycount 
				from (select count(avatarid) as scount from ".wtw_tableprefix."avatars where deleted=0) a1
					left join (select count(avatarid) as mycount 
						from ".wtw_tableprefix."avatars 
						where createuserid='".$wtwconnect->userid."' and deleted=0) a2 on 1=1
				union
			select '3D Models' as item, m1.scount, m2.mycount 
				from (select count(uploadobjectid) as scount from ".wtw_tableprefix."uploadobjects where deleted=0) m1
					left join (select count(uploadobjectid) as mycount 
						from ".wtw_tableprefix."uploadobjects 
						where (createuserid='".$wtwconnect->userid."' or userid='".$wtwconnect->userid."') and deleted=0) m2 on 1=1
				union
			select '3D Plugins' as item, p1.scount, p2.mycount 
				from (select count(pluginname) as scount from ".wtw_tableprefix."plugins where deleted=0) p1
					left join (select count(pluginname) as mycount 
						from ".wtw_tableprefix."plugins 
						where createuserid='".$wtwconnect->userid."' and deleted=0) p2 on 1=1
				union
			select 'Uploads' as item, u1.scount, u2.mycount 
				from (select count(uploadid) as scount from ".wtw_tableprefix."uploads where deleted=0) u1
					left join (select count(uploadid) as mycount 
						from ".wtw_tableprefix."uploads 
						where (createuserid='".$wtwconnect->userid."' or userid='".$wtwconnect->userid."') and deleted=0) u2 on 1=1
				union
			select 'Users 3D Avatars' as item, ua1.scount, ua2.mycount 
				from (select count(useravatarid) as scount from ".wtw_tableprefix."useravatars where deleted=0) ua1
					left join (select count(useravatarid) as mycount 
						from ".wtw_tableprefix."useravatars 
						where createuserid='".$wtwconnect->userid."' and deleted=0) ua2 on 1=1
				union
			select 'Users with Roles' as item, count(u2.userid) as scount, '' as mycount 
				from ".wtw_tableprefix."users u2 
					inner join (select userid 
						from ".wtw_tableprefix."usersinroles 
						where deleted=0 group by userid) ur2 on u2.userid=ur2.userid where u2.deleted=0
				union
			select 'Total User Accounts' as item, count(tu1.userid) as scount, '' as mycount 
				from ".wtw_tableprefix."users tu1 where tu1.deleted=0;			
		");		

		foreach ($zresults as $zrow) {
			if (empty($i)) {
				$zresponse[$i] = array(
					'item'=> $zrow["item"],
					'mycount'=> $zrow["mycount"],
					'scount'=> $zrow["scount"],
					'downloads'=> $zdownloads
				);
			} else {
				$zresponse[$i] = array(
					'item'=> $zrow["item"],
					'mycount'=> $zrow["mycount"],
					'scount'=> $zrow["scount"],
					'downloads'=> null
				);
			}
			$i += 1;
		}		
		$zresponse[$i] = array(
			'item'=> 'Total Folder Size',
			'mycount'=> $zwebsitesize,
			'scount'=> $zwebsitesize,
			'downloads'=> null
		);
		$i += 1;
	} else {
		/* all other users based on user/host account */
		
		/* check download queue for any pending */
		$i = 0;
		$zresults = $wtwconnect->query("
			select * 
				from ".wtw_tableprefix."downloads 
				where deleted=0 
					and (userip='".$wtwconnect->userip."'
						or downloaduserid='".$wtwconnect->userid."'
						or createuserid='".$wtwconnect->userid."')
				order by createdate desc, downloadid desc;");
		foreach ($zresults as $zrow) {
			$zdownloads[$i] = array(
				"downloadid" => $zrow["downloadid"],
				"webid" => $zrow["webid"],
				"webtype" => $zrow["webtype"],
				"userip" => $zrow["userip"],
				"fromurl" => $zrow["fromurl"],
				"createdate" => $zrow["createdate"]
			);
			$i += 1;
		}
		
		/* get server totals */
		$i = 0;
		$zresults = $wtwconnect->query("
			select '3D Communities' as item, c1.scount, c2.mycount 
				from (select count(communityid) as scount from ".wtw_tableprefix."communities where deleted=0) c1 
					left join (select count(communityid) as mycount 
						from ".wtw_tableprefix."communities 
						where createuserid='".$wtwconnect->userid."' and deleted=0) c2 on 1=1
				union
			select '3D Buildings' as item, b1.scount, b2.mycount 
				from (select count(buildingid) as scount from ".wtw_tableprefix."buildings where deleted=0) b1
					left join (select count(buildingid) as mycount 
						from ".wtw_tableprefix."buildings 
						where createuserid='".$wtwconnect->userid."' and deleted=0) b2 on 1=1
				union
			select '3D Things' as item, t1.scount, t2.mycount 
				from (select count(thingid) as scount from ".wtw_tableprefix."things where deleted=0) t1
					left join (select count(thingid) as mycount 
						from ".wtw_tableprefix."things 
						where createuserid='".$wtwconnect->userid."' and deleted=0) t2 on 1=1
				union
			select '3D Avatars' as item, a1.scount, a2.mycount 
				from (select count(avatarid) as scount from ".wtw_tableprefix."avatars where deleted=0) a1
					left join (select count(avatarid) as mycount 
						from ".wtw_tableprefix."avatars 
						where (hostuserid='".$wtwconnect->userid."'
							or createuserid='".$wtwconnect->userid."') and deleted=0) a2 on 1=1
				union
			select '3D Models' as item, m1.scount, m2.mycount 
				from (select count(uploadobjectid) as scount from ".wtw_tableprefix."uploadobjects where deleted=0) m1
					left join (select count(uploadobjectid) as mycount 
						from ".wtw_tableprefix."uploadobjects 
						where (createuserid='".$wtwconnect->userid."' or userid='".$wtwconnect->userid."') and deleted=0) m2 on 1=1
				union
			select 'Uploads' as item, u1.scount, u2.mycount 
				from (select count(uploadid) as scount from ".wtw_tableprefix."uploads where deleted=0) u1
					left join (select count(uploadid) as mycount 
						from ".wtw_tableprefix."uploads 
						where (createuserid='".$wtwconnect->userid."' or userid='".$wtwconnect->userid."') and deleted=0) u2 on 1=1
				union
			select 'Users 3D Avatars' as item, ua1.scount, ua2.mycount 
				from (select count(useravatarid) as scount from ".wtw_tableprefix."useravatars where deleted=0) ua1
					left join (select count(useravatarid) as mycount 
						from ".wtw_tableprefix."useravatars 
						where createuserid='".$wtwconnect->userid."' and deleted=0) ua2 on 1=1
		");		

		foreach ($zresults as $zrow) {
			/* calculate user drive space */
			$zfoldersize = 0;
			switch ($zrow["item"]) {
				case '3D Communities':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."communities 
							where deleted=0 
								and createuserid='".$wtwconnect->userid."';");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath."/content/uploads/communities/".$zrow2["communityid"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath."/content/uploads/communities/".$zrow2["communityid"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case '3D Buildings':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."buildings 
							where deleted=0 
								and createuserid='".$wtwconnect->userid."';");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath."/content/uploads/buildings/".$zrow2["buildingid"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath."/content/uploads/buildings/".$zrow2["buildingid"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case '3D Things':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."things 
							where deleted=0 
								and createuserid='".$wtwconnect->userid."';");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath."/content/uploads/things/".$zrow2["thingid"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath."/content/uploads/things/".$zrow2["thingid"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case '3D Avatars':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."avatars 
							where deleted=0 
								and (hostuserid='".$wtwconnect->userid."'
									or createuserid='".$wtwconnect->userid."');");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath."/content/uploads/avatars/".$zrow2["avatarid"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath."/content/uploads/avatars/".$zrow2["avatarid"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case '3D Models':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."uploadobjects 
							where deleted=0 
								and (createuserid='".$wtwconnect->userid."'
									or userid='".$wtwconnect->userid."');");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath.$zrow2["objectfolder"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath.$zrow2["objectfolder"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case 'Uploads':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."uploads 
							where deleted=0 
								and (createuserid='".$wtwconnect->userid."'
									or userid='".$wtwconnect->userid."');");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath.$zrow2["filepath"])) {
							$zsize = filesize($wtwconnect->rootpath.$zrow2["filepath"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
				case 'Users 3D Avatars':
					$zresults2 = $wtwconnect->query("
						select * 
							from ".wtw_tableprefix."useravatars 
							where deleted=0 
								and createuserid='".$wtwconnect->userid."';");
					foreach ($zresults2 as $zrow2) {
						if (file_exists($wtwconnect->rootpath."/content/uploads/useravatars/".$zrow2["useravatarid"])) {
							$zsize = $wtwconnect->dirSize($wtwconnect->rootpath."/content/uploads/useravatars/".$zrow2["useravatarid"]);
							$zwebsitesize += $zsize;
							$zfoldersize += $zsize;
						}
					}
					break;
			}
			/* add values to response array */
			if (empty($i)) {
				$zresponse[$i] = array(
					'item'=> $zrow["item"],
					'mycount'=> $zrow["mycount"],
					'scount'=> $zrow["scount"],
					'foldersize'=> $zfoldersize,
					'downloads'=> $zdownloads
				);
			} else {
				$zresponse[$i] = array(
					'item'=> $zrow["item"],
					'mycount'=> $zrow["mycount"],
					'scount'=> $zrow["scount"],
					'foldersize'=> $zfoldersize,
					'downloads'=> null
				);
			}
			$i += 1;
		}		
		
		$zresponse[$i] = array(
			'item'=> 'Total Folder Size',
			'mycount'=> '',
			'scount'=> $zwebsitesize,
			'foldersize'=> $zwebsitesize,
			'downloads'=> null
		);
		$i += 1;		
		
	}
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-dashboard.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: domaininfo.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides start position and scene data information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/domaininfo.php");
	
	/* get values from querystring or session */
	$zdomainname = $wtwconnect->getVal('domainname','');
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zcommunity = $wtwconnect->getVal('community','');
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zbuilding = $wtwconnect->getVal('building','');
	$zthingid = $wtwconnect->getVal('thingid','');
	$zthing = $wtwconnect->getVal('thing','');
	$zuserid = $wtwconnect->userid;
	$zconnectinggridid = "";
	$zdomaininfo = array();
	$zbuildinginfo = array();
	$zcommunityinfo = array();
	$zstartlocation = array();
	$zspawnzones = array();

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	if ((!isset($zcommunityid) || empty($zcommunityid)) && isset($zcommunity) && !empty($zcommunity)) {
		/* select communityid for community by url segment (pubname) */
		$zresults = $wtwconnect->query("
			select communityid 
				from ".wtw_tableprefix."webaliases 
				where communitypublishname='".$zcommunity."' 
					and deleted=0 
				order by communityid desc 
				limit 1;");
		foreach ($zresults as $zrow) {
			$zcommunityid = $zrow["communityid"];
		}
	}
	if ((!isset($zbuildingid) || empty($zbuildingid)) && isset($zbuilding) && !empty($zbuilding)) {
		/* select buildingid for community by url segment (pubname) */
		$zresults = $wtwconnect->query("
			select buildingid 
				from ".wtw_tableprefix."webaliases 
				where buildingpublishname = '".$zbuilding."' 
					and deleted=0 
				order by buildingid desc 
				limit 1;");
		foreach ($zresults as $zrow) {
			$zbuildingid = $zrow["buildingid"];
		}
	}
	if ($wtwconnect->hasValue($zcommunityid) && $wtwconnect->hasValue($zbuildingid)) {
		/* select connectinggridid */
		$zresults = $wtwconnect->query("
			select connectinggridid 
				from ".wtw_tableprefix."connectinggrids 
				where parentwebid='".$zcommunityid."' 
					and childwebid='".$zbuildingid."' 
					and deleted=0 
				limit 1;");
		foreach ($zresults as $zrow) {
			$zconnectinggridid = $zrow["connectinggridid"];
		}
		if (!isset($zconnectinggridid) || empty($zconnectinggridid)) {
			$zbuildingid = "";
		}
	}
	if (!isset($zcommunityid) || empty($zcommunityid)) {
		$zcommunityid = "";
	}
	if (!isset($zbuildingid) || empty($zbuildingid)) {
		$zbuildingid = "";
	}
	$zresults = array();
	if ($wtwconnect->hasValue($zcommunityid) && $wtwconnect->hasValue($zbuildingid)) {
		/* get domain info and connecting grids by communityid and buildingid */
		$zresults = $wtwconnect->query("
			select  c1.communityid,
				case when (select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where communityid=c1.communityid 
								and deleted=0 and not communityid='') is null then ''
					else
						(select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where communityid=c1.communityid 
								and deleted=0 and not communityid='')
					end as communityaccess,
				b1.buildingid,
				'' as thingid,
				case when (select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where buildingid=b1.buildingid 
								and deleted=0 and not buildingid='') is null then ''
					else
						(select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where buildingid=b1.buildingid 
								and deleted=0 and not buildingid='')
					end as buildingaccess,
				b1.positionx as positionx,
				b1.positiony as positiony,
				b1.positionz as positionz,
				b1.scalingx as scalingx,
				b1.scalingy as scalingy,
				b1.scalingz as scalingz,
				b1.rotationx as rotationx,
				b1.rotationy as rotationy,
				b1.rotationz as rotationz,
				connectinggrids.positionx as bcpositionx,
				connectinggrids.positiony as bcpositiony,
				connectinggrids.positionz as bcpositionz,
				connectinggrids.scalingx as bcscalingx,
				connectinggrids.scalingy as bcscalingy,
				connectinggrids.scalingz as bcscalingz,
				connectinggrids.rotationx as bcrotationx,
				connectinggrids.rotationy as bcrotationy,
				connectinggrids.rotationz as bcrotationz,
				c1.userid,
				c1.spawnactionzoneid,
				c1.gravity,
				c1.communityname as sitename,
				b1.buildingname,
				c1.communityname,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.textureid,
				case when c1.textureid = '' then ''
					else
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=c1.textureid limit 1)
					end as texturepath,
				c1.skydomeid,
				case when c1.skydomeid = '' then ''
					else
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=c1.skydomeid limit 1)
					end as skydomepath,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				'' as thingauthorizationid
			from (select * from ".wtw_tableprefix."communities 
						where deleted=0 and communityid = '".$zcommunityid."') c1
				left join (select * from ".wtw_tableprefix."connectinggrids 
						where deleted=0 and childwebid = '".$zbuildingid."' 
							and parentwebid = '".$zcommunityid."') connectinggrids
					on c1.communityid = connectinggrids.parentwebid
				left join (select * from ".wtw_tableprefix."buildings 
						where deleted=0 
							and buildingid = '".$zbuildingid."') b1 
					on b1.buildingid = connectinggrids.childwebid
				;");
	} else if ($wtwconnect->hasValue($zcommunityid)) {
		/* get domain info and connecting grids by communityid */
		$zresults = $wtwconnect->query("
			select c1.communityid,
				case when (select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where communityid=c1.communityid 
								and deleted=0 and not communityid='') is null then ''
					else
						(select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where communityid=c1.communityid 
								and deleted=0 and not communityid='')
					end as communityaccess,
				'' as buildingid,
				'' as thingid,
				'' as buildingaccess,
				c1.positionx as positionx,
				c1.positiony as positiony,
				c1.positionz as positionz,
				c1.scalingx as scalingx,
				c1.scalingy as scalingy,
				c1.scalingz as scalingz,
				c1.rotationx as rotationx,
				c1.rotationy as rotationy,
				c1.rotationz as rotationz,
				0 as bcpositionx,
				0 as bcpositiony,
				0 as bcpositionz,
				1 as bcscalingx,
				1 as bcscalingy,
				1 as bcscalingz,
				0 as bcrotationx,
				0 as bcrotationy,
				0 as bcrotationz,
				c1.userid,
				c1.spawnactionzoneid,
				c1.gravity,
				c1.communityname as sitename,
				'' as buildingname,
				c1.communityname,
				c1.groundpositiony,
				c1.waterpositiony,
				c1.textureid,
				case when c1.textureid = '' then ''
					else
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=c1.textureid limit 1)
					end as texturepath,
				c1.skydomeid,
				case when c1.skydomeid = '' then ''
					else
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=c1.skydomeid limit 1)
					end as skydomepath,
				c1.waterbumpheight,
				c1.watersubdivisions,
				c1.windforce,
				c1.winddirectionx,
				c1.winddirectiony,
				c1.winddirectionz,
				c1.waterwaveheight,
				c1.waterwavelength,
				c1.watercolorrefraction,
				c1.watercolorreflection,
				c1.watercolorblendfactor,
				c1.watercolorblendfactor2,
				c1.wateralpha,
				c1.sceneambientcolor,
				c1.sceneclearcolor,
				c1.sceneuseclonedmeshmap,
				c1.sceneblockmaterialdirtymechanism,
				c1.scenefogenabled,
				c1.scenefogmode,
				c1.scenefogdensity,
				c1.scenefogstart,
				c1.scenefogend,
				c1.scenefogcolor,
				c1.sundirectionalintensity,
				c1.sundiffusecolor,
				c1.sunspecularcolor,
				c1.sungroundcolor,
				c1.sundirectionx,
				c1.sundirectiony,
				c1.sundirectionz,
				c1.sunpositionx,
				c1.sunpositiony,
				c1.sunpositionz,
				c1.backlightintensity,
				c1.backlightdirectionx,
				c1.backlightdirectiony,
				c1.backlightdirectionz,
				c1.backlightpositionx,
				c1.backlightpositiony,
				c1.backlightpositionz,
				c1.backlightdiffusecolor,
				c1.backlightspecularcolor,
				c1.skytype,
				c1.skysize,
				c1.skyboxfolder,
				c1.skyboxfile,
				c1.skyboximageleft,
				c1.skyboximageup,
				c1.skyboximagefront,
				c1.skyboximageright,
				c1.skyboximagedown,
				c1.skyboximageback,
				c1.skypositionoffsetx,
				c1.skypositionoffsety,
				c1.skypositionoffsetz,
				c1.skyboxmicrosurface,
				c1.skyboxpbr,
				c1.skyboxasenvironmenttexture,
				c1.skyboxblur,
				c1.skyboxdiffusecolor,
				c1.skyboxspecularcolor,
				c1.skyboxambientcolor,
				c1.skyboxemissivecolor,
				c1.skyinclination,
				c1.skyluminance,
				c1.skyazimuth,
				c1.skyrayleigh,
				c1.skyturbidity,
				c1.skymiedirectionalg,
				c1.skymiecoefficient,
				'' as thingauthorizationid,
				'' as userauthorizationid
			from (select * from ".wtw_tableprefix."communities 
						where deleted=0 and communityid='".$zcommunityid."') c1;	
		");	
	} else if ($wtwconnect->hasValue($zbuildingid)) {
		/* select domain info and connecting grids by buildingid */
		$zresults = $wtwconnect->query("
			select '' as communityid,
				'' as communityaccess,
				b1.buildingid,
				'' as thingid,
				case when (select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where buildingid=b1.buildingid 
								and deleted=0 and not buildingid='') is null then ''
					else
						(select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where buildingid=b1.buildingid 
								and deleted=0 and not buildingid='')
					end as buildingaccess,
				b1.positionx as positionx,
				b1.positiony as positiony,
				b1.positionz as positionz,
				b1.scalingx as scalingx,
				b1.scalingy as scalingy,
				b1.scalingz as scalingz,
				b1.rotationx as rotationx,
				b1.rotationy as rotationy,
				b1.rotationz as rotationz,
				0 as bcpositionx,
				0 as bcpositiony,
				0 as bcpositionz,
				1 as bcscalingx,
				1 as bcscalingy,
				1 as bcscalingz,
				0 as bcrotationx,
				0 as bcrotationy,
				0 as bcrotationz,
				b1.userid,
				b1.spawnactionzoneid,
				b1.gravity,
				b1.buildingname as sitename,
				b1.buildingname,
				'default' as communityname,
				0 as groundpositiony,
				-50 as waterpositiony,
				'2391f1v9om09am77' as textureid,
				'/content/system/stock/dirt-512x512.jpg' as texturepath,
				'' as skydomeid,
				'' as skydomepath,
				.60 as waterbumpheight,
				2.00 as watersubdivisions,
				-10.00 as windforce,
				1.00 as winddirectionx,
				0.00 as winddirectiony,
				1.00 as winddirectionz,
				.20 as waterwaveheight,
				.02 as waterwavelength,
				'#23749c' as watercolorrefraction,
				'#52bcf1' as watercolorreflection,
				.20 as watercolorblendfactor,
				.20 as watercolorblendfactor2,
				.90 as wateralpha,
				'#E5E8E8' as sceneambientcolor,
				'#000000' as sceneclearcolor,
				1 as sceneuseclonedmeshmap,
				1 as sceneblockmaterialdirtymechanism,
				0 as scenefogenabled,
				'' as scenefogmode,
				0.01 as scenefogdensity,
				20 as scenefogstart,
				60 as scenefogend,
				'#c0c0c0' as scenefogcolor,
				1 as sundirectionalintensity,
				'#ffffff' as sundiffusecolor,
				'#ffffff' as sunspecularcolor,
				'#000000' as sungroundcolor,
				999 as sundirectionx,
				-999 as sundirectiony,
				999 as sundirectionz,
				0 as sunpositionx,
				1000 as sunpositiony,
				0 as sunpositionz,
				0.5 as backlightintensity,
				-999 as backlightdirectionx,
				999 as backlightdirectiony,
				-999 as backlightdirectionz,
				0 as backlightpositionx,
				1000 as backlightpositiony,
				0 as backlightpositionz,
				'#ffffff' as backlightdiffusecolor,
				'#ffffff' as backlightspecularcolor,
				'' as skytype,
				5000.00 as skysize,
				'' as skyboxfolder,
				'' as skyboxfile,
				'' as skyboximageleft,
				'' as skyboximageup,
				'' as skyboximagefront,
				'' as skyboximageright,
				'' as skyboximagedown,
				'' as skyboximageback,
				0 as skypositionoffsetx,
				0 as skypositionoffsety,
				0 as skypositionoffsetz,
				0 as skyboxmicrosurface,
				0 as skyboxpbr,
				0 as skyboxasenvironmenttexture,
				0 as skyboxblur,
				'#000000' as skyboxdiffusecolor,
				'#000000' as skyboxspecularcolor,
				'#000000' as skyboxambientcolor,
				'#000000' as skyboxemissivecolor,
				0 as skyinclination,
				1 as skyluminance,
				.25 as skyazimuth,
				2 as skyrayleigh,
				10 as skyturbidity,
				.8 as skymiedirectionalg,
				.005 as skymiecoefficient,
				'' as thingauthorizationid
			from (select * from ".wtw_tableprefix."buildings 
						where deleted=0 
							and buildingid='".$zbuildingid."') b1 
		;");
	} else if ($wtwconnect->hasValue($zthingid)) {
		/* select domain info and connecting grids by thingid */
		$zresults = $wtwconnect->query("
			select '' as communityid,
				'' as communityaccess,
				'' as buildingid,
				t1.thingid,
				case when (select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where thingid=t1.thingid 
								and deleted=0 and not thingid='') is null then ''
					else
						(select GROUP_CONCAT(userid) as useraccess 
							from ".wtw_tableprefix."userauthorizations 
							where thingid=t1.thingid 
								and deleted=0 and not thingid='')
					end as thingaccess,
				t1.positionx as positionx,
				t1.positiony as positiony,
				t1.positionz as positionz,
				t1.scalingx as scalingx,
				t1.scalingy as scalingy,
				t1.scalingz as scalingz,
				t1.rotationx as rotationx,
				t1.rotationy as rotationy,
				t1.rotationz as rotationz,
				0 as bcpositionx,
				0 as bcpositiony,
				0 as bcpositionz,
				1 as bcscalingx,
				1 as bcscalingy,
				1 as bcscalingz,
				0 as bcrotationx,
				0 as bcrotationy,
				0 as bcrotationz,
				t1.userid,
				t1.spawnactionzoneid,
				t1.gravity,
				t1.thingname as sitename,
				'' as buildingname,
				'default' as communityname,
				0 as groundpositiony,
				-50 as waterpositiony,
				'2391f1v9om09am77' as textureid,
				'/content/system/stock/dirt-512x512.jpg' as texturepath,
				'' as skydomeid,
				'' as skydomepath,
				.60 as waterbumpheight,
				2.00 as watersubdivisions,
				-10.00 as windforce,
				1.00 as winddirectionx,
				0.00 as winddirectiony,
				1.00 as winddirectionz,
				.20 as waterwaveheight,
				.02 as waterwavelength,
				'#23749c' as watercolorrefraction,
				'#52bcf1' as watercolorreflection,
				.20 as watercolorblendfactor,
				.20 as watercolorblendfactor2,
				.90 as wateralpha,
				'#E5E8E8' as sceneambientcolor,
				'#000000' as sceneclearcolor,
				1 as sceneuseclonedmeshmap,
				1 as sceneblockmaterialdirtymechanism,
				0 as scenefogenabled,
				'' as scenefogmode,
				0.01 as scenefogdensity,
				20 as scenefogstart,
				60 as scenefogend,
				'#c0c0c0' as scenefogcolor,
				1 as sundirectionalintensity,
				'#ffffff' as sundiffusecolor,
				'#ffffff' as sunspecularcolor,
				'#000000' as sungroundcolor,
				999 as sundirectionx,
				-999 as sundirectiony,
				999 as sundirectionz,
				0 as sunpositionx,
				1000 as sunpositiony,
				0 as sunpositionz,
				0.5 as backlightintensity,
				-999 as backlightdirectionx,
				999 as backlightdirectiony,
				-999 as backlightdirectionz,
				0 as backlightpositionx,
				1000 as backlightpositiony,
				0 as backlightpositionz,
				'#ffffff' as backlightdiffusecolor,
				'#ffffff' as backlightspecularcolor,
				'' as skytype,
				5000.00 as skysize,
				'' as skyboxfolder,
				'' as skyboxfile,
				'' as skyboximageleft,
				'' as skyboximageup,
				'' as skyboximagefront,
				'' as skyboximageright,
				'' as skyboximagedown,
				'' as skyboximageback,
				0 as skypositionoffsetx,
				0 as skypositionoffsety,
				0 as skypositionoffsetz,
				0 as skyboxmicrosurface,
				0 as skyboxpbr,
				0 as skyboxasenvironmenttexture,
				0 as skyboxblur,
				'#000000' as skyboxdiffusecolor,
				'#000000' as skyboxspecularcolor,
				'#000000' as skyboxambientcolor,
				'#000000' as skyboxemissivecolor,
				0 as skyinclination,
				1 as skyluminance,
				.25 as skyazimuth,
				2 as skyrayleigh,
				10 as skyturbidity,
				.8 as skymiedirectionalg,
				.005 as skymiecoefficient,
				'' as thingauthorizationid
			from (select * from ".wtw_tableprefix."things 
						where deleted=0 
							and thingid='".$zthingid."') t1 
		;");
	}

	$zazresults = array();
	$zspawnzones = array();
	if (!empty($zcommunityid)) {
		$zazresults = $wtwdb->query("
			select distinct az1.* 
			from ".wtw_tableprefix."actionzones az1 
			where (az1.communityid='".$zcommunityid."' 
				and az1.actionzonetype='spawnzone' 
				and az1.deleted=0)

			union
			select distinct az2.* 
			from ".wtw_tableprefix."actionzones az2
				inner join ".wtw_tableprefix."connectinggrids cg2
					on az2.buildingid=cg2.childwebid
					and cg2.childwebtype='building'
					and cg2.parentwebid='".$zcommunityid."'
					and cg2.parentwebtype='community'
			where (az2.actionzonetype='spawnzone' 
				and az2.deleted=0
				and cg2.deleted=0)

			union
			select distinct az3.* 
			from ".wtw_tableprefix."actionzones az3
				inner join ".wtw_tableprefix."connectinggrids cg3
					on az3.buildingid=cg3.childwebid
					and cg3.childwebtype='thing'
					and cg3.parentwebid='".$zcommunityid."'
					and cg3.parentwebtype='community'
			where (az3.actionzonetype='spawnzone' 
				and az3.deleted=0
				and cg3.deleted=0);			
		");		
	} else if (!empty($zbuildingid)) {
		$zazresults = $wtwdb->query("
			select distinct az1.* 
			from ".wtw_tableprefix."actionzones az1 
			where (az1.buildingid='".$zbuildingid."' 
				and az1.actionzonetype='spawnzone' 
				and az1.deleted=0)

			union
			select distinct az2.* 
			from ".wtw_tableprefix."actionzones az2
				inner join ".wtw_tableprefix."connectinggrids cg2
					on az2.thingid=cg2.childwebid
					and cg2.childwebtype='thing'
					and cg2.parentwebid='".$zbuildingid."'
					and cg2.parentwebtype='building'
			where (az2.actionzonetype='spawnzone' 
				and az2.deleted=0
				and cg2.deleted=0);			
		");
	} else if (!empty($zthingid)) {
		$zazresults = $wtwdb->query("
			select distinct az1.* 
			from ".wtw_tableprefix."actionzones az1 
			where (az1.thingid='".$zthingid."' 
				and az1.actionzonetype='spawnzone' 
				and az1.deleted=0);			
		");
	}
	if (count($zazresults) > 0) {
		$zspawnindex = 0;
		foreach ($zazresults as $zazrow) {
			$zspawnzones[$zspawnindex] = array(
				'actionzoneid'=>$zazrow["actionzoneid"],
				'communityid'=>$zazrow["communityid"],
				'buildingid'=>$zazrow["buildingid"],
				'thingid'=>$zazrow["thingid"],
				'loadactionzoneid'=>$zazrow["loadactionzoneid"],
				'actionzonename'=>$zazrow["actionzonename"],
				'actionzoneshape'=>$zazrow["actionzoneshape"],
				'actionzonetype'=>$zazrow["actionzonetype"],
				'positionx'=>$zazrow["positionx"],
				'positiony'=>$zazrow["positiony"],
				'positionz'=>$zazrow["positionz"],
				'scalingx'=>$zazrow["scalingx"],
				'scalingy'=>$zazrow["scalingy"],
				'scalingz'=>$zazrow["scalingz"],
				'rotationx'=>$zazrow["rotationx"],
				'rotationy'=>$zazrow["rotationy"],
				'rotationz'=>$zazrow["rotationz"]
			);
			$zspawnindex += 1;
		}
	}

	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zcommunityid = $zrow["communityid"];
		$zbuildingid = $zrow["buildingid"];
		$zthingid = $zrow["thingid"];
		$zdomaininfo = array(
			'communityid' => $zrow["communityid"],
			'buildingid' => $zrow["buildingid"],
			'thingid' => $zrow["thingid"],
			'sitename' => $zrow["sitename"],
			'spawnactionzoneid' => $zrow["spawnactionzoneid"],
			'gravity' => $zrow["gravity"],
			'userid' => $zrow["userid"]);
		$zbuildinginfo = array(
			'buildingid' => $zrow["buildingid"],
			'buildingname' => $zrow["buildingname"],
			'access' => $zrow["buildingaccess"]
		);	
		$zcommunityinfo = array(
			'communityid' => $zrow["communityid"],
			'communityname' => $zrow["communityname"],
			'access' => $zrow["communityaccess"],
			'textureid' => $zrow["textureid"],
			'texturepath' => $zrow["texturepath"],
			'skydomeid' => $zrow["skydomeid"],
			'skydomepath' => $zrow["skydomepath"],
			'waterbumpheight' => $zrow["waterbumpheight"],
			'watersubdivisions' => $zrow["watersubdivisions"],
			'windforce' => $zrow["windforce"],
			'winddirectionx' => $zrow["winddirectionx"],
			'winddirectiony' => $zrow["winddirectiony"],
			'winddirectionz' => $zrow["winddirectionz"],
			'waterwaveheight' => $zrow["waterwaveheight"],
			'waterwavelength' => $zrow["waterwavelength"],
			'watercolorrefraction' => $zrow["watercolorrefraction"],
			'watercolorreflection' => $zrow["watercolorreflection"],
			'watercolorblendfactor' => $zrow["watercolorblendfactor"],
			'watercolorblendfactor2' => $zrow["watercolorblendfactor2"],
			'wateralpha' => $zrow["wateralpha"],
			'sceneambientcolor' => $zrow["sceneambientcolor"],
			'sceneclearcolor' => $zrow["sceneclearcolor"],
			'sceneuseclonedmeshmap' => $zrow["sceneuseclonedmeshmap"],
			'sceneblockmaterialdirtymechanism' => $zrow["sceneblockmaterialdirtymechanism"],
			'sundirectionalintensity' => $zrow["sundirectionalintensity"],
			'sundiffusecolor' => $zrow["sundiffusecolor"],
			'sunspecularcolor' => $zrow["sunspecularcolor"],
			'sungroundcolor' => $zrow["sungroundcolor"],
			'sundirectionx' => $zrow["sundirectionx"],
			'sundirectiony' => $zrow["sundirectiony"],
			'sundirectionz' => $zrow["sundirectionz"],
			'sunpositionx' => $zrow["sunpositionx"],
			'sunpositiony' => $zrow["sunpositiony"],
			'sunpositionz' => $zrow["sunpositionz"],
			'backlightintensity' => $zrow["backlightintensity"],
			'backlightdirectionx' => $zrow["backlightdirectionx"],
			'backlightdirectiony' => $zrow["backlightdirectiony"],
			'backlightdirectionz' => $zrow["backlightdirectionz"],
			'backlightpositionx' => $zrow["backlightpositionx"],
			'backlightpositiony' => $zrow["backlightpositiony"],
			'backlightpositionz' => $zrow["backlightpositionz"],
			'backlightdiffusecolor' => $zrow["backlightdiffusecolor"],
			'backlightspecularcolor' => $zrow["backlightspecularcolor"],
			'scenefogenabled' => $zrow["scenefogenabled"],
			'scenefogmode' => $zrow["scenefogmode"],
			'scenefogdensity' => $zrow["scenefogdensity"],
			'scenefogstart' => $zrow["scenefogstart"],
			'scenefogend' => $zrow["scenefogend"],
			'scenefogcolor' => $zrow["scenefogcolor"],
			'skytype' => $zrow["skytype"],
			'skysize' => $zrow["skysize"],
			'skyboxfolder' => $zrow["skyboxfolder"],
			'skyboxfile' => $zrow["skyboxfile"],
			'skyboximageleft' => $zrow["skyboximageleft"],
			'skyboximageup' => $zrow["skyboximageup"],
			'skyboximagefront' => $zrow["skyboximagefront"],
			'skyboximageright' => $zrow["skyboximageright"],
			'skyboximagedown' => $zrow["skyboximagedown"],
			'skyboximageback' => $zrow["skyboximageback"],
			'skypositionoffsetx' => $zrow["skypositionoffsetx"],
			'skypositionoffsety' => $zrow["skypositionoffsety"],
			'skypositionoffsetz' => $zrow["skypositionoffsetz"],
			'skyboxmicrosurface' => $zrow["skyboxmicrosurface"],
			'skyboxpbr' => $zrow["skyboxpbr"],
			'skyboxasenvironmenttexture' => $zrow["skyboxasenvironmenttexture"],
			'skyboxblur' => $zrow["skyboxblur"],
			'skyboxdiffusecolor' => $zrow["skyboxdiffusecolor"],
			'skyboxspecularcolor' => $zrow["skyboxspecularcolor"],
			'skyboxambientcolor' => $zrow["skyboxambientcolor"],
			'skyboxemissivecolor' => $zrow["skyboxemissivecolor"],
			'skyinclination' => $zrow["skyinclination"],
			'skyluminance' => $zrow["skyluminance"],
			'skyazimuth' => $zrow["skyazimuth"],
			'skyrayleigh' => $zrow["skyrayleigh"],
			'skyturbidity' => $zrow["skyturbidity"],
			'skymiedirectionalg' => $zrow["skymiedirectionalg"],
			'skymiecoefficient' => $zrow["skymiecoefficient"]
		);	
		$zposition = array(
			'x' => $zrow["positionx"],
			'y' => $zrow["positiony"],
			'z' => $zrow["positionz"],
			'groundpositiony' => $zrow["groundpositiony"],
			'waterpositiony' => $zrow["waterpositiony"]
		);	
		$zscaling = array(
			'x' => $zrow["scalingx"],
			'y' => $zrow["scalingy"],
			'z' => $zrow["scalingz"]
		);	
		$zrotation = array(
			'x' => $zrow["rotationx"],
			'y' => $zrow["rotationy"],
			'z' => $zrow["rotationz"]
		);	
		$zstartlocation = array(
			'position' => $zposition,
			'scaling' => $zscaling,
			'rotation' => $zrotation);
	}
	$zresponse['domaininfo'] = $zdomaininfo;
	$zresponse['buildinginfo'] = $zbuildinginfo;
	$zresponse['communityinfo'] = $zcommunityinfo;
	$zresponse['startlocation'] = $zstartlocation;
	$zresponse['spawnzones'] = $zspawnzones;
	$zresponse['serverfranchiseid'] = '';
	$zresponse['useraccesslist'] = null; /* getaccesslist("", $zbuildingid, $zcommunityid); */
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-domaininfo.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: franchises.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web aliases information for franchises */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/franchises.php");
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	
	$zresults = array();
	
	/* get web aliases for a user */
	$zresults = $wtwconnect->query("
		select w1.*,
			c1.communityname,
			b1.buildingname,
			t1.thingname
		from ".wtw_tableprefix."webaliases w1
			left join ".wtw_tableprefix."communities c1
				on w1.communityid=c1.communityid
			left join ".wtw_tableprefix."buildings b1
				on w1.buildingid=b1.buildingid
			left join ".wtw_tableprefix."things t1
				on w1.thingid=t1.thingid
		where w1.deleted=0
			and w1.franchise=1
		order by 
			w1.hostuserid desc,
			w1.domainname,
			w1.communitypublishname,
			w1.buildingpublishname,
			w1.thingpublishname,
			w1.communityid,
			w1.buildingid,
			w1.thingid,
			w1.webaliasid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$webalias = array(
			'serverfranchiseid' => '',
			'webaliasid' => $zrow["webaliasid"],
			'domainname' => $zrow["domainname"],
			'webalias' => $zrow["webalias"],
			'communityid' => $zrow["communityid"],
			'communitypublishname' => $zrow["communitypublishname"],
			'communityname' => $zrow["communityname"],
			'buildingid' => $zrow["buildingid"],
			'buildingpublishname' => $zrow["buildingpublishname"],
			'buildingname' => $zrow["buildingname"],
			'thingid' => $zrow["thingid"],
			'thingpublishname' => $zrow["thingpublishname"],
			'thingname' => $zrow["thingname"],
			'forcehttps' => $zrow["forcehttps"],
			'franchise' => $zrow["franchise"],
			'franchiseid' => $zrow["franchiseid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"]);
		$zresponse[$i] = $webalias;
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-franchises.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: moldsbywebid.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides molds information by 3D Community, Building, or Thing */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/moldsbywebid.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zthingid = $wtwconnect->getVal('thingid','');
	$zactionzoneid = $wtwconnect->getVal('actionzoneid','');
	$zactionzoneind = $wtwconnect->getVal('actionzoneind','-1');
	$zparentactionzoneind = $wtwconnect->getVal('parentactionzoneind','-1');
	$zparentname = $wtwconnect->getVal('parentname','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');
	$zforcegraphiclevel = $wtwconnect->getVal('graphiclevel','-1');

	/* gets all related molds */
	$zresults = $wtwconnect->query("
		select distinct
			a1.communitymoldid,
			'' as buildingmoldid,
			'' as thingmoldid,
			a1.communitymoldid as moldid,
			a1.communityid,
			'' as buildingid,
			'' as thingid,
			'' as altconnectinggridid,
			a1.loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
			a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			communities.analyticsid as communityanalyticsid,
			communities.communityname,
			communities.snapshotid as communitysnapshotid,
            case when communities.snapshotid is null or communities.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=communities.snapshotid limit 1)
				end as communitysnapshoturl,
			'' as buildinganalyticsid,
			'' as buildingname,
			'' as buildingsnapshotid,
			'' as buildingsnapshoturl,
			'' as thinganalyticsid,
			'' as thingname,
			'' as thingsnapshotid,
			'' as thingsnapshoturl,
			a1.alttag as communityalttag,
			'' as buildingalttag,
			'' as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."communitymolds 
				where communityid='".$zcommunityid."' 
					and csgmoldid=a1.communitymoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		from ".wtw_tableprefix."communitymolds a1 
			inner join (select * from ".wtw_tableprefix."actionzones 
					where communityid='".$zcommunityid."' 
						and (not communityid='') and deleted=0) a2
				on a1.loadactionzoneid = a2.actionzoneid
				or a1.unloadactionzoneid = a2.actionzoneid
				or a1.actionzoneid = a2.actionzoneid
			left join (select * 
					from ".wtw_tableprefix."communities 
					where communityid='".$zcommunityid."' and deleted=0) communities
				on a1.communityid =  communities.communityid
		where a1.deleted=0
		
		union all
		
		select distinct 
			'' as communitymoldid,
			a1.buildingmoldid,
			'' as thingmoldid,
			a1.buildingmoldid as moldid,
			'' as communityid,
			a1.buildingid,
			'' as thingid,
			'' as altconnectinggridid,
			a1.loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
									where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
									where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
									where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
			a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			'' as communityanalyticsid,
			'' as communityname,
			'' as communitysnapshotid,
			'' as communitysnapshoturl,
			buildings.analyticsid as buildinganalyticsid,
			buildings.buildingname,
			buildings.snapshotid as buildingsnapshotid,
            case when buildings.snapshotid is null or buildings.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=buildings.snapshotid limit 1)
				end as buildingsnapshoturl,
			'' as thinganalyticsid,
			'' as thingname,
			'' as thingsnapshotid,
			'' as thingsnapshoturl,
			'' as communityalttag,
			a1.alttag as buildingalttag,
			'' as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."buildingmolds 
				where buildingid='".$zbuildingid."' and csgmoldid=a1.buildingmoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		from ".wtw_tableprefix."buildingmolds a1 
			inner join (select * from ".wtw_tableprefix."actionzones 
					where buildingid='".$zbuildingid."' and (not buildingid='') and deleted=0) a2
				on a1.loadactionzoneid = a2.actionzoneid
				or a1.unloadactionzoneid = a2.actionzoneid
				or a1.actionzoneid = a2.actionzoneid
			left join (select * 
					from ".wtw_tableprefix."buildings 
					where buildingid='".$zbuildingid."' and deleted=0) buildings
				on a1.buildingid =  buildings.buildingid
		where a1.deleted=0
			
		union all

		select distinct 
			'' as communitymoldid,
			'' as buildingmoldid,
			a1.thingmoldid,
			a1.thingmoldid as moldid,
			'' as communityid,
			'' as buildingid,
			a1.thingid,
			'' as altconnectinggridid,
			a1.loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
			a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			'' as communityanalyticsid,
			'' as communityname,
			'' as communitysnapshotid,
			'' as communitysnapshoturl,
			'' as buildinganalyticsid,
			'' as buildingname,
			'' as buildingsnapshotid,
			'' as buildingsnapshoturl,
			things.analyticsid as thinganalyticsid,
			things.thingname,
			things.snapshotid as thingsnapshotid,
            case when things.snapshotid is null or things.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=things.snapshotid limit 1)
				end as thingsnapshoturl,
			'' as communityalttag,
			'' as buildingalttag,
			a1.alttag as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."thingmolds 
				where thingid='".$zthingid."' and csgmoldid=a1.thingmoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		from ".wtw_tableprefix."thingmolds a1 
			inner join (select * 
					from ".wtw_tableprefix."actionzones 
					where thingid='".$zthingid."' 
						and (not thingid='') and deleted=0) a2
				on a1.loadactionzoneid = a2.actionzoneid
				or a1.unloadactionzoneid = a2.actionzoneid
				or a1.actionzoneid = a2.actionzoneid
			left join (select * 
					from ".wtw_tableprefix."things 
					where thingid='".$zthingid."' and deleted=0) things
				on a1.thingid =  things.thingid
		where a1.deleted=0
	 
		union all
		
		select distinct 
			'' as communitymoldid,
			'' as buildingmoldid,
			a1.thingmoldid,
			a1.thingmoldid as moldid,
			'' as communityid,
			'' as buildingid,
			a1.thingid,
			connectinggrids.connectinggridid as altconnectinggridid,
			connectinggrids.altloadactionzoneid as loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 	
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 	
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
			a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			'' as communityanalyticsid,
			'' as communityname,
			'' as communitysnapshotid,
			'' as communitysnapshoturl,
			'' as buildinganalyticsid,
			'' as buildingname,
			'' as buildingsnapshotid,
			'' as buildingsnapshoturl,
			things.analyticsid as thinganalyticsid,
			things.thingname,
			things.snapshotid as thingsnapshotid,
            case when things.snapshotid is null or things.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=things.snapshotid limit 1)
				end as thingsnapshoturl,
			'' as communityalttag,
			'' as buildingalttag,
			a1.alttag as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."thingmolds 
				where thingid='".$zthingid."' 
					and csgmoldid=a1.thingmoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		from (select * from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zbuildingid."' 
						and (not parentwebid='') 
						and parentwebtype='building' 
						and childwebtype='thing' 
						and (not altloadactionzoneid='') 
						and deleted=0) connectinggrids
			inner join ".wtw_tableprefix."thingmolds a1
				on connectinggrids.childwebid = a1.thingid
			left join (select * 
					from ".wtw_tableprefix."things where deleted=0) things
				on connectinggrids.childwebid =  things.thingid
		where a1.deleted=0

		union all
		
		select distinct 
			'' as communitymoldid,
			'' as buildingmoldid,
			a1.thingmoldid,
			a1.thingmoldid as moldid,
			'' as communityid,
			'' as buildingid,
			a1.thingid,
			connectinggrids.connectinggridid as altconnectinggridid,
			connectinggrids.altloadactionzoneid as loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
			a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			'' as communityanalyticsid,
			'' as communityname,
			'' as communitysnapshotid,
			'' as communitysnapshoturl,
			'' as buildinganalyticsid,
			'' as buildingname,
			'' as buildingsnapshotid,
			'' as buildingsnapshoturl,
			things.analyticsid as thinganalyticsid,
			things.thingname,
			things.snapshotid as thingsnapshotid,
            case when things.snapshotid is null or things.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=things.snapshotid limit 1)
				end as thingsnapshoturl,
			'' as communityalttag,
			'' as buildingalttag,
			a1.alttag as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."thingmolds 
					where thingid='".$zthingid."' 
						and csgmoldid=a1.thingmoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		FROM (select * from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zcommunityid."' 
						and (not parentwebid='') 
						and parentwebtype='community' 
						and childwebtype='thing' 
						and (not altloadactionzoneid='') 
						and deleted=0) connectinggrids
			inner join ".wtw_tableprefix."thingmolds a1
				on connectinggrids.childwebid = a1.thingid
			left join (select * 
					from ".wtw_tableprefix."things where deleted=0) things
				on connectinggrids.childwebid =  things.thingid
		where a1.deleted=0
		
		union all
		
		select distinct 
			'' as communitymoldid,
			'' as buildingmoldid,
			a1.thingmoldid,
			a1.thingmoldid as moldid,
			'' as communityid,
			'' as buildingid,
			a1.thingid,
			connectinggrids.connectinggridid as altconnectinggridid,
			connectinggrids.altloadactionzoneid as loadactionzoneid,
			a1.unloadactionzoneid,
			a1.shape,
			a1.covering,
			a1.positionx,
			a1.positiony,
			a1.positionz,
			a1.scalingx,
			a1.scalingy,
			a1.scalingz,
			a1.rotationx,
			a1.rotationy,
			a1.rotationz,
			a1.special1,
			a1.special2,
			a1.uoffset,
			a1.voffset,
			a1.uscale,
			a1.vscale,
            a1.uploadobjectid,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
            case when '".$zforcegraphiclevel."' = '1' then 1
				else
					case when '".$zforcegraphiclevel."' = '0' then 0
						else a1.graphiclevel
					end 
				end as graphiclevel,
			case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.textureid limit 1)
					end 
				end as textureid,
            case when a1.textureid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumpid,
            case when a1.texturebumpid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.heightmapid limit 1)
					end 
				end as heightmapid,
            case when a1.heightmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.mixmapid limit 1)
					end 
				end as mixmapid,
            case when a1.mixmapid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturerid limit 1)
					end 
				end as texturerid,
            case when a1.texturerid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturegid limit 1)
					end 
				end as texturegid,
            case when a1.texturegid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebid limit 1)
					end 
				end as texturebid,
            case when a1.texturebid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprid,
            case when a1.texturebumprid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgid,
            case when a1.texturebumpgid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbid,
            case when a1.texturebumpbid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
            a1.videoid,
            case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads 
								where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
            case when a1.videoposterid = '' then ''
				else
					case when (a1.graphiclevel = '1' and not '".$zforcegraphiclevel."' = '0') or '".$zforcegraphiclevel."' = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
            a1.soundid,
            case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
            a1.soundname,
            a1.soundattenuation,
            a1.soundloop,
            a1.soundmaxdistance,
            a1.soundrollofffactor,
            a1.soundrefdistance,
            a1.soundconeinnerangle,
            a1.soundconeouterangle,
            a1.soundconeoutergain,
			a1.diffusecolor,
			a1.emissivecolor,
			a1.specularcolor,
			a1.ambientcolor,
			a1.webtext,
			a1.webstyle,
			a1.opacity,
            a1.sideorientation,
            a1.billboard,
			a1.waterreflection,
            a1.receiveshadows,
            a1.castshadows,
			a1.subdivisions,
			a1.minheight,
			a1.maxheight,
			a1.checkcollisions,
			a1.ispickable,
			a1.actionzoneid,
			a1.actionzone2id,
			a1.csgmoldid,
			a1.csgaction,
			a1.physicsenabled,
			a1.physicscenterx,
			a1.physicscentery,
			a1.physicscenterz,
			a1.physicsextentsx,
			a1.physicsextentsy,
			a1.physicsextentsz,
			a1.physicsfriction,
			a1.physicsistriggershape,
			a1.physicsmass,
			a1.physicspointax,
			a1.physicspointay,
			a1.physicspointaz,
			a1.physicspointbx,
			a1.physicspointby,
			a1.physicspointbz,
			a1.physicsradius,
			a1.physicsrestitution,
			a1.physicsrotationw,
			a1.physicsrotationx,
			a1.physicsrotationy,
			a1.physicsrotationz,
			a1.physicsstartasleep,
			a1.createdate,
			a1.createuserid,
			a1.updatedate,
			a1.updateuserid,
			'' as communityanalyticsid,
			'' as communityname,
			'' as communitysnapshotid,
			'' as communitysnapshoturl,
			'' as buildinganalyticsid,
			'' as buildingname,
			'' as buildingsnapshotid,
			'' as buildingsnapshoturl,
			things.analyticsid as thinganalyticsid,
			things.thingname,
			things.snapshotid as thingsnapshotid,
            case when things.snapshotid is null or things.snapshotid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=things.snapshotid limit 1)
				end as thingsnapshoturl,
			'' as communityalttag,
			'' as buildingalttag,
			a1.alttag as thingalttag,
            a1.jsfunction,
            a1.jsparameters,
            (select count(*) from ".wtw_tableprefix."thingmolds 
					where thingid='".$zthingid."' 
						and csgmoldid=a1.thingmoldid) as csgcount,
            case when a1.shape = 'terrain' then 10
				when a1.shape = 'floor' then 9
                else
					0
				end as sortorder
		from (select * from ".wtw_tableprefix."connectinggrids 
					where parentwebid='".$zcommunityid."' 
						and (not parentwebid='') 
						and parentwebtype='community' 
						and childwebtype='building' 
						and (not altloadactionzoneid='') 
						and deleted=0) connectinggrids
			inner join ".wtw_tableprefix."thingmolds a1
				on connectinggrids.childwebid = a1.thingid
			left join (select * 
					from ".wtw_tableprefix."things 
					where deleted=0) things
				on connectinggrids.childwebid =  things.thingid
		where a1.deleted=0

		order by 
			 sortorder desc, csgmoldid desc, moldid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$zmolds = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zobjectanimations = null;
		$ztempwebtext = '';
		$zinloadactionzone = '0';
		if ($zactionzoneid == $zrow["loadactionzoneid"] && $zactionzoneid != "") {
			$zinloadactionzone = '1';
		}
		if ($wtwconnect->hasValue($zrow["webtext"])) {
			$ztempwebtext = implode('',(array)$zrow["webtext"]);
		}
		if ($wtwconnect->hasValue($zrow["uploadobjectid"])) {
			$zobjectanimations = $wtwconnect->getobjectanimations($zrow["uploadobjectid"]);
		}
		$zcommunityinfo = array(
			'communityid'=> $zrow["communityid"],
			'communityind'=> '',
			'communityname'=> $wtwconnect->escapeHTML($zrow["communityname"]),
			'snapshotid' => $zrow["communitysnapshotid"],
			'snapshoturl' => $zrow["communitysnapshoturl"],
			'analyticsid'=> $zrow["communityanalyticsid"]
		);
		$zbuildinginfo = array(
			'buildingid'=> $zrow["buildingid"],
			'buildingind'=> '',
			'buildingname'=> $wtwconnect->escapeHTML($zrow["buildingname"]),
			'snapshotid' => $zrow["buildingsnapshotid"],
			'snapshoturl' => $zrow["buildingsnapshoturl"],
			'analyticsid'=> $zrow["buildinganalyticsid"]
		);
		$zthinginfo = array(
			'thingid'=> $zrow["thingid"],
			'thingind'=> '',
			'thingname'=> $wtwconnect->escapeHTML($zrow["thingname"]),
			'snapshotid' => $zrow["thingsnapshotid"],
			'snapshoturl' => $zrow["thingsnapshoturl"],
			'analyticsid'=> $zrow["thinganalyticsid"]
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"],
			'scroll'=>''
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"],
			'special1'=> $zrow["special1"],
			'special2'=> $zrow["special2"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"],
			'billboard'=> $zrow["billboard"]
		);
		$zcsg = array(
			'moldid'=> $zrow["csgmoldid"], 
			'moldind'=>'-1',
			'action'=> $zrow["csgaction"], 
			'count'=> $zrow["csgcount"] 
		);
		$zobjects = array(
			'uploadobjectid'=> $zrow["uploadobjectid"], 
			'folder'=> $zrow["objectfolder"], 
			'file'=> $zrow["objectfile"],
			'objectanimations'=> $zobjectanimations,
			'light'=> '',
			'shadows'=> ''
		);
		$zgraphics = array(
			'texture'=> array(
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'bumpid'=> $zrow["texturebumpid"],
				'bumppath'=> $zrow["texturebumppath"],
				'videoid'=> $zrow["videoid"],
				'video'=> $zrow["video"],
				'videoposterid'=> $zrow["videoposterid"],
				'videoposter'=> $zrow["videoposter"],
				'backupid'=> '',
				'loaded'=> '0'
			),
			'heightmap'=> array(
				'original'=> $zrow["heightmappath"],
				'id'=> $zrow["heightmapid"],
				'path'=> $zrow["heightmappath"],
				'minheight'=> $zrow["minheight"],
				'maxheight'=> $zrow["maxheight"],
				'mixmapid'=> $zrow["mixmapid"],
				'mixmappath'=> $zrow["mixmappath"],
				'texturerid'=> $zrow["texturerid"],
				'texturerpath'=> $zrow["texturerpath"],
				'texturegid'=> $zrow["texturegid"],
				'texturegpath'=> $zrow["texturegpath"],
				'texturebid'=> $zrow["texturebid"],
				'texturebpath'=> $zrow["texturebpath"],
				'texturebumprid'=> $zrow["texturebumprid"],
				'texturebumprpath'=> $zrow["texturebumprpath"],
				'texturebumpgid'=> $zrow["texturebumpgid"],
				'texturebumpgpath'=> $zrow["texturebumpgpath"],
				'texturebumpbid'=> $zrow["texturebumpbid"],
				'texturebumpbpath'=> $zrow["texturebumpbpath"]
			),
			'uoffset'=> $zrow["uoffset"],
			'voffset'=> $zrow["voffset"],
			'uscale'=> $zrow["uscale"],
			'vscale'=> $zrow["vscale"],
			'level'=> $zrow["graphiclevel"],
			'receiveshadows'=> $zrow["receiveshadows"],
			'castshadows'=> $zrow["castshadows"],
			'waterreflection'=> $zrow["waterreflection"], 
			'webimages'=> $wtwconnect->getwebimages($zrow["thingmoldid"], $zrow["buildingmoldid"], $zrow["communitymoldid"], $zrow["graphiclevel"])
		);
		$zwebtext = array(
			'webtext'=> $zrow["webtext"],
			'fullheight'=> '0',
			'scrollpos'=> '0',
			'webstyle'=> $zrow["webstyle"]
		);
		$zalttag = array(
			'name'=> ''
		);
		$zpaths = array(
			'path1'=> $wtwconnect->getmoldpoints($zrow["thingmoldid"], $zrow["buildingmoldid"], $zrow["communitymoldid"], 1, $zrow["shape"]),
			'path2'=> $wtwconnect->getmoldpoints($zrow["thingmoldid"], $zrow["buildingmoldid"], $zrow["communitymoldid"], 2, $zrow["shape"])
		);
		$zcolor = array(
			'diffusecolor'=> $zrow["diffusecolor"],
			'emissivecolor'=> $zrow["emissivecolor"],
			'specularcolor'=> $zrow["specularcolor"],
			'ambientcolor'=> $zrow["ambientcolor"]
		);
		$zsound = array(
			'id' => $zrow["soundid"],
			'path' => $zrow["soundpath"],
			'name' => $zrow["soundname"],
			'attenuation' => $zrow["soundattenuation"],
			'loop' => $zrow["soundloop"],
			'maxdistance' => $zrow["soundmaxdistance"],
			'rollofffactor' => $zrow["soundrollofffactor"],
			'refdistance' => $zrow["soundrefdistance"],
			'coneinnerangle' => $zrow["soundconeinnerangle"],
			'coneouterangle' => $zrow["soundconeouterangle"],
			'coneoutergain' => $zrow["soundconeoutergain"],
			'sound' => ''
		);
		$zphysics = array(
			'enabled'=>$zrow["physicsenabled"],
			'center'=>array(
				'x'=>$zrow["physicscenterx"],
				'y'=>$zrow["physicscentery"],
				'z'=>$zrow["physicscenterz"]
			),
			'extents'=>array(
				'x'=>$zrow["physicsextentsx"],
				'y'=>$zrow["physicsextentsy"],
				'z'=>$zrow["physicsextentsz"]
			),
			'friction'=>$zrow["physicsfriction"],
			'istriggershape'=>$zrow["physicsistriggershape"],
			'mass'=>$zrow["physicsmass"],
			'pointa'=>array(
				'x'=>$zrow["physicspointax"],
				'y'=>$zrow["physicspointay"],
				'z'=>$zrow["physicspointaz"]
			),
			'pointb'=>array(
				'x'=>$zrow["physicspointbx"],
				'y'=>$zrow["physicspointby"],
				'z'=>$zrow["physicspointbz"]
			),
			'radius'=>$zrow["physicsradius"],
			'restitution'=>$zrow["physicsrestitution"],
			'rotation'=>array(
				'x'=>$zrow["physicsrotationx"],
				'y'=>$zrow["physicsrotationy"],
				'z'=>$zrow["physicsrotationz"],
				'w'=>$zrow["physicsrotationw"]
			),
			'startasleep'=>$zrow["physicsstartasleep"]
		);



		if ($wtwconnect->hasValue($zrow['communitymoldid'])) {
			$zalttag = array(
				'name'=> $zrow['communityalttag']
			);
		}
		if ($wtwconnect->hasValue($zrow['buildingmoldid'])) {
			$zalttag = array(
				'name'=> $zrow['buildingalttag']
			);
		}
		if ($wtwconnect->hasValue($zrow['thingmoldid'])) {
			$zalttag = array(
				'name'=> $zrow['thingalttag']
			);
		}
		$zmolds[$i] = array(
			'communityinfo'=> $zcommunityinfo, 
			'buildinginfo'=> $zbuildinginfo, 
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'moldid'=> $zrow["moldid"], 
			'moldind'=> '-1',
			'shape'=> $zrow["shape"], 
			'covering'=> $zrow["covering"], 
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'csg'=> $zcsg,
			'objects'=> $zobjects,
			'graphics'=> $zgraphics, 
			'webtext'=> $zwebtext, 
			'alttag'=> $zalttag,
			'paths'=> $zpaths,
			'color'=> $zcolor,
			'sound'=> $zsound,
			'physics'=> $zphysics,
			'subdivisions'=> $zrow["subdivisions"], 
			'subdivisionsshown'=>'0',
			'shown'=>'0',
			'opacity'=> $zrow["opacity"], 
			'checkcollisions'=> $zrow["checkcollisions"], 
			'ispickable'=> $zrow["ispickable"], 
			'jsfunction'=> $zrow["jsfunction"],
			'jsparameters'=> $zrow["jsparameters"],
			'actionzoneid'=> $zrow["actionzoneid"],
			'actionzoneind'=> $zactionzoneind,
			'actionzone2id'=> $zrow["actionzone2id"],
			'actionzone2ind'=> '-1',
			'parentactionzoneind'=> $zparentactionzoneind,
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'loadactionzoneind'=> '-1',
			'unloadactionzoneid'=> $zrow["unloadactionzoneid"],
			'unloadactionzoneind'=> '-1',
			'inloadactionzone'=> $zinloadactionzone,
			'altconnectinggridid'=> $zrow["altconnectinggridid"],
			'altconnectinggridind'=> '-1',
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'attachmoldind'=> '-1',
			'loaded'=> '0',
			'parentname'=> $zparentname,
			'moldname'=> '');
		$i += 1;
	}
	$zresponse['molds'] = $zmolds;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-moldsbywebid.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: objectanimations.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides animations for 3D Models */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/objectanimations.php");
	
	/* get values from querystring or session */
	$zuploadobjectid = $wtwconnect->getVal('uploadobjectid','');

	/* select object animation definitions */
	$zresults = $wtwconnect->query("
		select a1.*,
			case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
			case when a1.moldevent='onload' then '2'
				when a1.moldevent='' then '0'
				else '1'
			end as sorder
		from ".wtw_tableprefix."uploadobjectanimations a1
		where a1.uploadobjectid='".$zuploadobjectid."'
			and a1.deleted=0
		order by sorder, a1.moldevent, a1.animationname, a1.objectanimationid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$zobjectanimations = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zobjectanimations[$i] = array(
			'objectanimationid'=> $zrow['objectanimationid'],
			'animationname'=> $zrow['animationname'],
			'moldevent'=> $zrow['moldevent'],
			'moldnamepart'=> $zrow['moldnamepart'],
			'startframe'=> $zrow['startframe'],
			'endframe'=> $zrow['endframe'],
			'animationloop'=> $zrow['animationloop'],
			'speedratio'=> $zrow['speedratio'],
			'additionalscript'=> $zrow['additionalscript'],
			'additionalparameters'=> $zrow['additionalparameters'],
			'animationendscript'=> $zrow['animationendscript'],
			'animationendparameters'=> $zrow['animationendparameters'],
			'stopcurrentanimations'=> $zrow['stopcurrentanimations'],
			'soundid'=> $zrow['soundid'],
			'soundpath'=> $zrow['soundpath'],
			'soundmaxdistance'=> $zrow['soundmaxdistance']
		);
		$i += 1;
	}
	$zresponse['objectanimations'] = $zobjectanimations;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-objectanimations.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: pluginsrequired.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Community information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

	function getPluginPHP($zcontentpath, $zpluginphp, $zfolder) {
		/* check for initial php file under plugin folder (name should match the folder with .php added) */
		global $wtwconnect; 
		$zresponse = array(
			'pluginname' => '',
			'version' => '0.0.0',
			'latestversion' => '0.0.0',
			'title' => '',
			'author' => '',
			'description' => '',
			'foldername' => $zfolder,
			'filename' => $zpluginphp,
			'imageurl' => '',
			'updatedate' => '',
			'updateurl' => '',
			'loaded' => '1',
			'active' => '0',
			'required' => '0',
			'optional' => '0'
		);
		try {
			$i = 0;
			if (file_exists($zpluginphp)) {
				$zpluginname = "";
				$zlines = file($zpluginphp);
				foreach ($zlines as $zline) {
					$zline = str_replace("\n","",str_replace("\r","",$zline));
					if (strpos($zline,'=') !== false) {
						$zlineparts = explode("=",$zline);
						if ($zlineparts[0] != null && $zlineparts[1] != null) {
							$zpart = strtolower(trim(str_replace("#","",$zlineparts[0])));
							$zvalue = trim(str_replace("#","",$zlineparts[1]));
							switch ($zpart) {
								case "pluginname":
									$zpluginname = $zvalue;
									$zvalue = strtolower($zvalue);
								case "version":
									$zvalue = strtolower($zvalue);
									$zresponse["latestversion"] = $zvalue;
								case "title":
								case "description":
								case "author":
									$zresponse[$zpart] = $zvalue;
									$i += 1;
									break;
							}
						}
					}
				}
				if ($wtwconnect->hasValue($zpluginname)) {
					$zresponse['active'] = getPluginActive($zpluginname);
				}
			}
			if (file_exists($zcontentpath.'/plugins/'.$zfolder.'/'.$zfolder.'.png')) {
				$zresponse['imageurl'] = '/content/plugins/'.$zfolder.'/'.$zfolder.'.png';
			} else if (file_exists($zcontentpath.'/plugins/'.$zfolder.'/'.$zfolder.'.jpg')) {
				$zresponse['imageurl'] = '/content/plugins/'.$zfolder.'/'.$zfolder.'.jpg';
			} else if (file_exists($zcontentpath.'/plugins/'.$zfolder.'/'.$zfolder.'.gif')) {
				$zresponse['imageurl'] = '/content/plugins/'.$zfolder.'/'.$zfolder.'.gif';
			} else {
				$zresponse['imageurl'] = '/content/system/images/plugin.png';
			}
		} catch (Exception $e) {
			$wtwconnect->serror("connect-pluginsrequired.php-getPluginPHP=".$e->getMessage());
		}
		return $zresponse;
	}
	
	function getPluginActive($zpluginname) {
		/* see if a plugin is set to active */
		global $wtwconnect;
		$zactive = "0";
		try {
			$zresponse = $wtwconnect->query("
				select active
				from ".wtw_tableprefix."plugins
				where lower(pluginname)=lower('".$zpluginname."')
					and deleted=0;");
			foreach ($zresponse as $zrow) {
				if (!empty($zrow["active"]) & isset($zrow["active"])) {
					if ($zrow["active"] == "1") {
						$zactive = "1";
					}
				}
			}
		} catch (Exception $e) {
			$wtwconnect->serror("connect-pluginsrequired.php-getPluginActive=".$e->getMessage());
		}
		return $zactive;
	}
	
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/pluginsrequired.php");
	
	/* get values from querystring or session */
	$zwebid = $wtwconnect->getVal('webid','');
	$zwebtype = $wtwconnect->getVal('webtype','');
	
	$zplugins = array();
	
	$i = 0;
	$zfilepath = $wtwconnect->contentpath."/plugins";
	if (file_exists($zfilepath)) {
		$zfolders = new DirectoryIterator($zfilepath);
		foreach ($zfolders as $zfileinfo) {
			if ($zfileinfo->isDir() && !$zfileinfo->isDot()) {
				$zfolder = $zfileinfo->getFilename();
				$zpluginphp = $zfilepath."/".$zfolder."/".$zfolder.".php";
				$zplugins[$i] = getPluginPHP($wtwconnect->contentpath, $zpluginphp, $zfolder);
				$zrequired = '0';
				$zoptional = '0';
				$zresults = $wtwconnect->query("
					select *
					from ".wtw_tableprefix."pluginsrequired
					where deleted=0
						and pluginname='".$zfolder."'
						and webid='".$zwebid."'
						and webtype='".$zwebtype."';");
				foreach ($zresults as $zrow) {
					$zrequired = '1';
					$zoptional = $zrow['optional'];
					if ($zoptional == '1') {
						$zrequired = '0';
					}
				}
				$zplugins[$i]["required"] = $zrequired;
				$zplugins[$i]["optional"] = $zoptional;
				$i += 1;
			}
		}
	}

	/* populate any missing webtype fields */
	$zresults = $wtwconnect->query("
		select *
		from ".wtw_tableprefix."pluginsrequired
		where deleted=0
			and webid='".$zwebid."';");
	foreach ($zresults as $zrow) {
		$zpluginname = $zrow['pluginname'];
		$zoptional = $zrow['optional'];
		$zfound = false;
		foreach ($zplugins as $zplugin) {
			if ($zpluginname == $zplugin['pluginname']) {
				$zfound = true;
			}
		}
		if ($zfound == false) {
			$zfilename = $zfilepath."/".$zpluginname."/".$zpluginname.".php";
			$zplugins[$i] = array(
				'pluginname' => $zpluginname,
				'version' => '0.0.0',
				'latestversion' => '0.0.0',
				'title' => '',
				'author' => '',
				'description' => '',
				'foldername' => $zpluginname,
				'filename' => $zfilename,
				'updatedate' => '',
				'updateurl' => '',
				'loaded' => '0',
				'active' => '0',
				'required' => '1',
				'optional' => $zoptional
			);
			$i += 1;
		}
	}
	
	/* sort the results by plugin name, then title */
	function arraysort($a, $b) {
		if ($a["pluginname"] == $b["pluginname"]) {
			return ($a["title"] > $b["title"]) ? 1 : -1;
		}
		return ($a["pluginname"] > $b["pluginname"]) ? 1 : -1;
	}
	usort($zplugins, "arraysort");

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	echo json_encode($zplugins);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-pluginsrequired.php=".$e->getMessage());
}
?>
----------------------
----------------------
File: rating.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Community information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/rating.php");
	
	/* get values from querystring or session */
	$zwebid = $wtwconnect->getVal('webid','');
	$zwebtype = $wtwconnect->getVal('webtype','');
	$zextended = $wtwconnect->getVal('extended','1');
	
	$zrating = 'Not Rated';
	$zratingvalue = 0;
	$zratingtext = 'This 3D Website has not been rated. Take caution.';
	$zcontentwarning = '';
	$zunratedcontent = '0';
	$zcontentrating = '';

	/* populate any missing webtype fields */
	$zresults = $wtwconnect->query("
		select cr1.*,
			c1.communityname,
			b1.buildingname,
			t1.thingname,
			a1.displayname as avatarname
		from ".wtw_tableprefix."contentratings cr1
			left join ".wtw_tableprefix."communities c1
				on c1.communityid=cr1.webid
			left join ".wtw_tableprefix."buildings b1
				on b1.buildingid=cr1.webid
			left join ".wtw_tableprefix."things t1
				on t1.thingid=cr1.webid
			left join ".wtw_tableprefix."avatars a1
				on a1.avatarid=cr1.webid
		where cr1.webtype='';");
	foreach ($zresults as $zrow) {
		$znewwebtype = '';
		if ($wtwconnect->hasValue($zrow["communityname"])) {
			$znewwebtype = 'community';
		} else if ($wtwconnect->hasValue($zrow["buildingname"])) {
			$znewwebtype = 'building';
		} else if ($wtwconnect->hasValue($zrow["thingname"])) {
			$znewwebtype = 'thing';
		} else if ($wtwconnect->hasValue($zrow["avatarname"])) {
			$znewwebtype = 'avatar';
		}
		$wtwconnect->query("
			update ".wtw_tableprefix."contentratings
			set webtype='".$znewwebtype."'
			where contentratingid='".$zrow["contentratingid"]."'
				and webtype=''
			limit 1;");
	}
	
	/* select base web content rating */
	$zresults = $wtwconnect->query("
		select cr1.*,
			c1.communityname,
			b1.buildingname,
			t1.thingname,
			a1.displayname as avatarname
		from ".wtw_tableprefix."contentratings cr1
			left join ".wtw_tableprefix."communities c1
				on c1.communityid=cr1.webid
			left join ".wtw_tableprefix."buildings b1
				on b1.buildingid=cr1.webid
			left join ".wtw_tableprefix."things t1
				on t1.thingid=cr1.webid
			left join ".wtw_tableprefix."avatars a1
				on a1.avatarid=cr1.webid
		where cr1.webid='".$zwebid."'
			and cr1.webtype='".$zwebtype."'
		order by cr1.createdate desc
		limit 1;");
	if (count($zresults) == 0) {
		$zunratedcontent = '1';
		$zcontentrating .= '<b>The main 3D Website is not Rated</b><br /><br />';
	} else {
		foreach ($zresults as $zrow) {
			$zsitename = '';
			if ($wtwconnect->hasValue($zrow["communityname"])) {
				$zsitename = $zrow["communityname"];
			} else if ($wtwconnect->hasValue($zrow["buildingname"])) {
				$zsitename = $zrow["buildingname"];
			} else if ($wtwconnect->hasValue($zrow["thingname"])) {
				$zsitename = $zrow["thingname"];
			} else if ($wtwconnect->hasValue($zrow["avatarname"])) {
				$zsitename = $zrow["avatarname"];
			}
			$zrating = $zrow["rating"];
			$zratingtext = $wtwconnect->getRatingText($zrating);
			if (is_numeric($zrow["ratingvalue"])) {
				$zratingvalue = (int)$zrow["ratingvalue"];
			}
			$zcontentrating .= "<b><span style='color:#FEFFCE'>".$zsitename."</span> 3D Website is Rated <span style='color:#FEFFCE'>".$zrating."</span></b>. ".$zratingtext."<br />";

			if ($wtwconnect->hasValue($zrow["contentwarning"])) {
				if ($zextended == '0') {
					$zcontentwarning = $zrow["contentwarning"];
				} else {
					$zcontentwarning .= $zrow["contentwarning"] . "<br /><br />";
				}
				$zcontentrating .= $zrow["contentwarning"] . "<br /><br />";
			} else {
				$zcontentrating .= "<br />";
			}
		}
	}
	
	if ($zextended == '1' && isset($zwebid) && !empty($zwebid)) {
		/* look for content ratings on child items */
		$zresults = $wtwconnect->query("
			select cg1.*,
				cr1.rating,
				cr1.ratingvalue,
				cr1.contentwarning,
				c1.communityname,
				b1.buildingname,
				t1.thingname,
				a1.displayname as avatarname
			from 
				(select parentwebid, parentwebtype, childwebid, childwebtype 
					from ".wtw_tableprefix."connectinggrids
					where deleted=0
					group by parentwebid, parentwebtype, childwebid, childwebtype) cg1
				left join ".wtw_tableprefix."contentratings cr1
					on cg1.childwebid = cr1.webid
				left join ".wtw_tableprefix."communities c1
					on c1.communityid=cr1.webid
				left join ".wtw_tableprefix."buildings b1
					on b1.buildingid=cr1.webid
				left join ".wtw_tableprefix."things t1
					on t1.thingid=cr1.webid
				left join ".wtw_tableprefix."avatars a1
					on a1.avatarid=cr1.webid
			where cg1.parentwebid='".$zwebid."';");
		foreach ($zresults as $zrow) {
			if (isset($zrow["ratingvalue"])) {
				$zsitename = '';
				if ($wtwconnect->hasValue($zrow["communityname"])) {
					$zsitename = $zrow["communityname"];
				} else if ($wtwconnect->hasValue($zrow["buildingname"])) {
					$zsitename = $zrow["buildingname"];
				} else if ($wtwconnect->hasValue($zrow["thingname"])) {
					$zsitename = $zrow["thingname"];
				} else if ($wtwconnect->hasValue($zrow["avatarname"])) {
					$zsitename = $zrow["avatarname"];
				}
				/* if rating is higher than base rating, update the base rating */
				if (is_numeric($zrow["ratingvalue"])) {
					if ((int)$zrow["ratingvalue"] > $zratingvalue) {
						$zrating = $zrow["rating"];
						$zratingvalue = (int)$zrow["ratingvalue"];
					}
					$zratingtext = $wtwconnect->getRatingText($zrow["rating"]);
					$zcontentrating .= "<b><span style='color:#FEFFCE'>".$zsitename."</span> 3D Website is Rated <span style='color:#FEFFCE'>".$zrow["rating"]."</span></b>. ".$zratingtext."<br />";
				} else {
					$zcontentrating .= "<b><span style='color:#FEFFCE'>".$zsitename."</span> 3D Website is Unrated</b><br />";
				}
				/* append any content warnings */
				if ($wtwconnect->hasValue($zrow["contentwarning"])) {
					$zcontentwarning .= $zrow["contentwarning"] . "<br /><br />";
					$zcontentrating .= $zrow["contentwarning"] . "<br /><br />";
				} else {
					$zcontentrating .= "<br />";
				}
			} else {
				$zunratedcontent = '1';
			}
			/* check for one more level deep - buildings with things in it */
			if ($zrow["childwebtype"] == 'building') {
				$zresults2 = $wtwconnect->query("
					select cg1.*,
						cr1.rating,
						cr1.ratingvalue,
						cr1.contentwarning,
						c1.communityname,
						b1.buildingname,
						t1.thingname,
						a1.displayname as avatarname
					from 
						(select parentwebid, parentwebtype, childwebid, childwebtype 
							from ".wtw_tableprefix."connectinggrids
							where deleted=0
							group by parentwebid, parentwebtype, childwebid, childwebtype) cg1
						left join ".wtw_tableprefix."contentratings cr1
							on cg1.childwebid = cr1.webid
						left join ".wtw_tableprefix."communities c1
							on c1.communityid=cr1.webid
						left join ".wtw_tableprefix."buildings b1
							on b1.buildingid=cr1.webid
						left join ".wtw_tableprefix."things t1
							on t1.thingid=cr1.webid
						left join ".wtw_tableprefix."avatars a1
							on a1.avatarid=cr1.webid
					where cg1.parentwebid='".$zrow["childwebid"]."';");
				foreach ($zresults2 as $zrow2) {
					if (isset($zrow2["ratingvalue"])) {
						$zsitename = '';
						if ($wtwconnect->hasValue($zrow2["communityname"])) {
							$zsitename = $zrow2["communityname"];
						} else if ($wtwconnect->hasValue($zrow2["buildingname"])) {
							$zsitename = $zrow2["buildingname"];
						} else if ($wtwconnect->hasValue($zrow2["thingname"])) {
							$zsitename = $zrow2["thingname"];
						} else if ($wtwconnect->hasValue($zrow["avatarname"])) {
							$zsitename = $zrow["avatarname"];
						}
						/* if rating is higher than base rating, update the base rating */
						if (is_numeric($zrow2["ratingvalue"])) {
							if ((int)$zrow2["ratingvalue"] > $zratingvalue) {
								$zrating = $zrow2["rating"];
								$zratingvalue = (int)$zrow2["ratingvalue"];
							}
							$zratingtext = $wtwconnect->getRatingText($zrow["rating"]);
							$zcontentrating .= "<b><span style='color:#FEFFCE'>".$zsitename."</span> 3D Website is Rated <span style='color:#FEFFCE'>".$zrow["rating"]."</span></b>. ".$zratingtext."<br />";
						} else {
							$zcontentrating .= "<b><span style='color:#FEFFCE'>".$zsitename."</span> 3D Website is Unrated</b><br />";
						}
						/* append any content warnings */
						if ($wtwconnect->hasValue($zrow2["contentwarning"])) {
							$zcontentwarning .= $zrow2["contentwarning"] . "<br /><br />";
							$zcontentrating .= $zrow2["contentwarning"] . "<br /><br />";
						} else {
							$zcontentrating .= "<br />";
						}
					} else {
						$zunratedcontent = '1';
					}
				}
			}
		}
		if ($zunratedcontent == '1') {
			$zcontentrating .= "<hr /><b><span style='color:#FFCECE'>*This 3D Website also contains Unrated Content. Take caution.</span></b><br /><br />";
		}
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$zratingtext = $wtwconnect->getRatingText($zrating);
	
	/* format the JSON response */
	$zresponse = array(
		'serverfranchiseid' => '',
		'rating' => $zrating,
		'ratingvalue' => $zratingvalue,
		'ratingtext' => $zratingtext,
		'contentrating' => base64_encode($zcontentrating),
		'contentwarning' => base64_encode($zcontentwarning),
		'unratedcontent' => $zunratedcontent
	);
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-rating.php=".$e->getMessage());
}
?>
----------------------
----------------------
File: roles.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides list of roles */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/roles.php");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zroles = array();
	
	if ($wtwconnect->hasPermission(array("admin"))) {
		/* get user roles information */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."roles
			where deleted=0
			order by rolename, roleid;");
		
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			$zroles[$i] = array(
				'roleid' => $zrow["roleid"],
				'rolename' => $zrow["rolename"],
				'createdate' => $zrow["createdate"]
				);
			$i += 1;
		}
	}
	echo json_encode($zroles);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-roles.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: scripts.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides added javascripts information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/scripts.php");
	
	/* get values from querystring or session */
	$zactionzoneid = $wtwconnect->getVal('actionzoneid','');
	
	$zscripts = array();
	if ($wtwconnect->hasPermission(array("admin"))) {
		$i = 0;
		/* get scripts related to community, building, or thing by action zone (loadzone) */
		$zresults = $wtwconnect->query("
			select *
			from ".wtw_tableprefix."scripts
			where deleted=0
				and actionzoneid='".$zactionzoneid."';");
		foreach ($zresults as $zrow) {
			$zscripts[$i] = array(
				'scriptid'=> $zrow["scriptid"],
				'scriptname'=> $zrow["scriptname"],
				'scriptpath'=> $zrow["scriptpath"]
			);
			$i += 1;
		}
	}
	echo json_encode($zscripts);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-scripts.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: share.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides shared 3D Community, 3D Building, and 3D Thing information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

$zuploads = array();
$zupload = 0;
$zuploadobjects = array();
$zuploadobject = 0;
$zscripts = array();
$zscript = 0;
$zavataranimations = array();
$zavataranimation = 0;
$zusers = array();
$zuser = 0;

function addUploadID($zuploadid, $zrecursive) {
	global $wtwconnect;
	try {
		global $zuploads;
		global $zupload;
		if ($wtwconnect->hasValue($zuploadid)) {
			$zfound = false;
			foreach ($zuploads as $zrowup) {
				if ($zrowup["uploadid"] == $zuploadid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select upload file data */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."uploads
					where uploadid='".$zuploadid."'
					and deleted=0;");
				foreach ($zresults as $zrow) {
					$zfilepath = $zrow["filepath"];
					if ($wtwconnect->hasValue($zfilepath)) {
						if (substr($zfilepath, 0, 4) != "http") {
							$zfilepath = $wtwconnect->domainurl.$zfilepath;
						}
					}
					$zuploads[$zupload] = array (
						'uploadid'=>$zrow["uploadid"],
						'pastuploadid'=>$zrow["pastuploadid"],
						'originalid'=>$zrow["originalid"],
						'websizeid'=>$zrow["websizeid"],
						'thumbnailid'=>$zrow["thumbnailid"],
						'userid'=>$zrow["userid"],
						'filetitle'=>$wtwconnect->escapeHTML($zrow["filetitle"]),
						'filename'=>$zrow["filename"],
						'fileextension'=>$zrow["fileextension"],
						'filesize'=>$zrow["filesize"],
						'filetype'=>$zrow["filetype"],
						'filepath'=>$zfilepath,
						'filedata'=>$zrow["filedata"],
						'imagewidth'=>$zrow["imagewidth"],
						'imageheight'=>$zrow["imageheight"],
						'stock'=>$zrow["stock"],
						'hidedate'=>$zrow["hidedate"],
						'hideuserid'=>$zrow["hideuserid"],
						'hide'=>$zrow["hide"],
						'checkeddate'=>$zrow["checkeddate"],
						'checkeduserid'=>$zrow["checkeduserid"],
						'checked'=>$zrow["checked"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"]
					);
					$zupload += 1;
					if ($zrecursive) {
						if ($zrow["originalid"] != $zuploadid) {
							addUploadID($zrow["originalid"], false);
						}
						if ($zrow["websizeid"] != $zuploadid) {
							addUploadID($zrow["websizeid"], false);
						}
						if ($zrow["thumbnailid"] != $zuploadid) {
							addUploadID($zrow["thumbnailid"], false);
						}
					}
					addUserID($zrow["userid"]);
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-share.php-addUploadID=".$e->getMessage());
	}
}

function addUploadObjectID($zuploadobjectid) {
	global $wtwconnect;
	try {
		global $zuploadobjects;
		global $zuploadobject;
		if ($wtwconnect->hasValue($zuploadobjectid)) {
			$zfound = false;
			foreach ($zuploadobjects as $zrowuo) {
				if ($zrowuo["uploadobjectid"] == $zuploadobjectid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select object upload data */
				$zresults = $wtwconnect->query("
					select * from ".wtw_tableprefix."uploadobjects
					where uploadobjectid='".$zuploadobjectid."'
						and deleted=0
					limit 1;");
				foreach ($zresults as $zrow) {
					/* get list of uploaded objects in folder */
					$zobjectfiles = array();
					if ($wtwconnect->hasValue($zrow["objectfolder"])) {
						$zfiles = 0;
						$zdir = str_replace('/content/',$wtwconnect->contentpath.'/',$zrow["objectfolder"]);
						$zdir = rtrim($zdir, "/");
						if (is_dir($zdir)) {
							if ($zdh = opendir($zdir)) {
								while (($zfile = readdir($zdh)) !== false) {
									if ($zfile != '.' && $zfile != '..') {
										$zobjectfiles[$zfiles] = array(
											'filename'=>$zfile,
											'filepath'=>$wtwconnect->domainurl.$zrow["objectfolder"].$zfile
											);
										$zfiles += 1;
									}
								}
								closedir($zdh);
							}
						}
					}
					
					/* get uploaded objects animations */
					$zresultsuoanim = $wtwconnect->query("
						select *
						from ".wtw_tableprefix."uploadobjectanimations
						where uploadobjectid='".$zrow["uploadobjectid"]."'
							and deleted=0;");
					$zuoanim = 0;
					$zuploadobjectanimations = array();
					foreach ($zresultsuoanim as $zrowuoanim) {
						$zuploadobjectanimations[$zuoanim] = array(
							'objectanimationid'=>$zrowuoanim["objectanimationid"],
							'uploadobjectid'=>$zrowuoanim["uploadobjectid"],
							'userid'=>$zrowuoanim["userid"],
							'animationname'=>$wtwconnect->escapeHTML($zrowuoanim["animationname"]),
							'moldnamepart'=>$wtwconnect->escapeHTML($zrowuoanim["moldnamepart"]),
							'moldevent'=>$zrowuoanim["moldevent"],
							'startframe'=>$zrowuoanim["startframe"],
							'endframe'=>$zrowuoanim["endframe"],
							'animationloop'=>$zrowuoanim["animationloop"],
							'speedratio'=>$zrowuoanim["speedratio"],
							'animationendscript'=>$zrowuoanim["animationendscript"],
							'animationendparameters'=>$zrowuoanim["animationendparameters"],
							'stopcurrentanimations'=>$zrowuoanim["stopcurrentanimations"],
							'additionalscript'=>$zrowuoanim["additionalscript"],
							'additionalparameters'=>$zrowuoanim["additionalparameters"],
							'soundid'=>$zrowuoanim["soundid"],
							'soundmaxdistance'=>$zrowuoanim["soundmaxdistance"],
							'createdate'=>$zrowuoanim["createdate"],
							'createuserid'=>$zrowuoanim["createuserid"],
							'updatedate'=>$zrowuoanim["updatedate"],
							'updateuserid'=>$zrowuoanim["updateuserid"]
						);
						$zuoanim += 1;
						addUploadID($zrowuoanim["soundid"], false);
						addUserID($zrowuoanim["userid"]);
						addUserID($zrowuoanim["createuserid"]);
						addUserID($zrowuoanim["updateuserid"]);
					}			
					
					/* set upload objects array */
					$zuploadobjects[$zuploadobject] = array (
						'uploadobjectid'=>$zrow["uploadobjectid"],
						'userid'=>$zrow["userid"],
						'objectfolder'=>$wtwconnect->domainurl.$zrow["objectfolder"],
						'objectfile'=>$zrow["objectfile"],
						'stock'=>$zrow["stock"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"],
						'objectfiles'=>$zobjectfiles,
						'uploadobjectanimations'=>$zuploadobjectanimations
					);
					$zuploadobject += 1;
					addUserID($zrow["userid"]);
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-share.php-addUploadObjectID=".$e->getMessage());
	}
}

function addScriptID($zscriptid, $zscripturl) {
	global $wtwconnect;
	try {
		global $zscripts;
		global $zscript;
		if ($wtwconnect->hasValue($zscriptid)) {
			$zfound = false;
			foreach ($zscripts as $zrowscript) {
				if ($zrowscript["scriptid"] == $zscriptid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select script */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."scripts
					where scriptid='".$zscriptid."'
					limit 1;");
				foreach ($zresults as $zrow) {
					$zscripts[$zscript] = array (
						'scriptid'=>$zrow["scriptid"],
						'pastscriptid'=>$zrow["pastscriptid"],
						'actionzoneid'=>$zrow["actionzoneid"],
						'webtype'=>$zrow["webtype"],
						'webid'=>$zrow["webid"],
						'scriptname'=>$zrow["scriptname"],
						'scriptfilename'=>$zrow["scriptpath"],
						'scriptpath'=>$zscripturl.$zrow["scriptpath"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"]
					);
					$zscript += 1;
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-share.php-addScriptID=".$e->getMessage());
	}
}

function addAvatarAnimationID($zavataranimationid) {
	global $wtwconnect;
	try {
		global $zavataranimations;
		global $zavataranimation;
		if ($wtwconnect->hasValue($zavataranimationid)) {
			$zfound = false;
			foreach ($zavataranimations as $zrowanim) {
				if ($zrowanim["avataranimationid"] == $zavataranimationid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select avatar animations */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."avataranimations
					where avataranimationid='".$zavataranimationid."'
					limit 1;");
				foreach ($zresults as $zrow) {
					$zanimationicon = '';
					if (!empty($zrow["animationicon"])) {
						$zanimationicon = $wtwconnect->domainurl.$zrow["animationicon"];
					}
					
					$zavataranimations[$zavataranimation] = array (
						'avataranimationid'=>$zrow["avataranimationid"],
						'pastavataranimationid'=>$zrow["pastavataranimationid"],
						'avatarid'=>$zrow["avatarid"],
						'userid'=>$zrow["userid"],
						'loadpriority'=>$zrow["loadpriority"],
						'animationevent'=>$zrow["animationevent"],
						'animationfriendlyname'=>$zrow["animationfriendlyname"],
						'animationicon'=>$zanimationicon,
						'objectfolder'=>$wtwconnect->domainurl.$zrow["objectfolder"],
						'objectfile'=>$zrow["objectfile"],
						'startframe'=>$zrow["startframe"],
						'endframe'=>$zrow["endframe"],
						'animationloop'=>$zrow["animationloop"],
						'speedratio'=>$zrow["speedratio"],
						'soundid'=>$zrow["soundid"],
						'soundmaxdistance'=>$zrow["soundmaxdistance"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"]
					);
					$zavataranimation += 1;
					addUploadID($zrow["soundid"], false);
					addUserID($zrow["userid"]);
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-share.php-addAvatarAnimationID=".$e->getMessage());
	}
}

function addUserID($zuserid) {
	global $wtwconnect;
	try {
		global $zusers;
		global $zuser;
		if ($wtwconnect->hasValue($zuserid)) {
			$zfound = false;
			foreach ($zusers as $zrowup) {
				if ($zrowup["userid"] == $zuserid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select user */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."users
					where userid='".$zuserid."'
					limit 1;");
				foreach ($zresults as $zrow) {
					$zusers[$zuser] = array (
						'userid'=>$zrow["userid"],
						'displayname'=>$wtwconnect->escapeHTML($zrow["displayname"]),
						'email'=>$zrow["email"],
						'uploadpathid'=>$zrow["uploadpathid"]
					);
					$zuser += 1;
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-share.php-addUserID=".$e->getMessage());
	}
}

try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/share.php");
	
	/* get values from querystring or session */
	$zwebid = $wtwconnect->getVal('webid','');
	$zwebtype = $wtwconnect->getVal('webtype','');
	$zuserid = $wtwconnect->getVal('userid','');
	$zsharehash = $wtwconnect->getVal('sharehash','');

	$zwebtypes = '';
	switch ($zwebtype) {
		case "community":
			$zwebtypes = 'communities';
			break;
		case "building":
			$zwebtypes = 'buildings';
			break;
		case "thing":
			$zwebtypes = 'things';
			break;
	}

	$zresponse = array();

	addUserID($zuserid);
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	if (!empty($zwebtypes)) {
		/* get 3D Web */
		$zresults = $wtwconnect->query("
			select *
			from ".wtw_tableprefix.$zwebtypes."
			where ".$zwebtype."id='".$zwebid."'
				and shareuserid='".$zuserid."'
				and sharehash='".$zsharehash."'
				and deleted=0
			limit 1;");
		
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			/* get parent connecting grids */
			$zresultscg = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."connectinggrids
				where childwebid='".$zwebid."'
					and childwebtype='".$zwebtype."'
					and parentwebid=''
					and deleted=0;");
			$zcg = 0;
			$zconnectinggrids = array();
			foreach ($zresultscg as $zrowcg) {
				$zconnectinggrids[$zcg] = array(
					'connectinggridid'=>$zrowcg["connectinggridid"],
					'pastconnectinggridid'=>$zrowcg["pastconnectinggridid"],
					'parentwebid'=>$zrowcg["parentwebid"],
					'parentwebtype'=>$zrowcg["parentwebtype"],
					'childwebid'=>$zrowcg["childwebid"],
					'childwebtype'=>$zrowcg["childwebtype"],
					'positionx'=>$zrowcg["positionx"],
					'positiony'=>$zrowcg["positiony"],
					'positionz'=>$zrowcg["positionz"],
					'scalingx'=>$zrowcg["scalingx"],
					'scalingy'=>$zrowcg["scalingy"],
					'scalingz'=>$zrowcg["scalingz"],
					'rotationx'=>$zrowcg["rotationx"],
					'rotationy'=>$zrowcg["rotationy"],
					'rotationz'=>$zrowcg["rotationz"],
					'loadactionzoneid'=>$zrowcg["loadactionzoneid"],
					'altloadactionzoneid'=>$zrowcg["altloadactionzoneid"],
					'unloadactionzoneid'=>$zrowcg["unloadactionzoneid"],
					'attachactionzoneid'=>$zrowcg["attachactionzoneid"],
					'alttag'=>$wtwconnect->escapeHTML($zrowcg["alttag"]),
					'createdate'=>$zrowcg["createdate"],
					'createuserid'=>$zrowcg["createuserid"],
					'updatedate'=>$zrowcg["updatedate"],
					'updateuserid'=>$zrowcg["updateuserid"]
				);
				$zcg += 1;
				addUserID($zrowcg["createuserid"]);
				addUserID($zrowcg["updateuserid"]);
			}

			/* get child connecting grids */
			$zresultscg = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."connectinggrids
				where parentwebid='".$zwebid."'
					and parentwebtype='".$zwebtype."'
					and deleted=0;");
			$zcg = 0;
			$zchildconnectinggrids = array();
			foreach ($zresultscg as $zrowcg) {
				$zchildconnectinggrids[$zcg] = array(
					'connectinggridid'=>$zrowcg["connectinggridid"],
					'pastconnectinggridid'=>$zrowcg["pastconnectinggridid"],
					'parentwebid'=>$zrowcg["parentwebid"],
					'parentwebtype'=>$zrowcg["parentwebtype"],
					'childwebid'=>$zrowcg["childwebid"],
					'childwebtype'=>$zrowcg["childwebtype"],
					'positionx'=>$zrowcg["positionx"],
					'positiony'=>$zrowcg["positiony"],
					'positionz'=>$zrowcg["positionz"],
					'scalingx'=>$zrowcg["scalingx"],
					'scalingy'=>$zrowcg["scalingy"],
					'scalingz'=>$zrowcg["scalingz"],
					'rotationx'=>$zrowcg["rotationx"],
					'rotationy'=>$zrowcg["rotationy"],
					'rotationz'=>$zrowcg["rotationz"],
					'loadactionzoneid'=>$zrowcg["loadactionzoneid"],
					'altloadactionzoneid'=>$zrowcg["altloadactionzoneid"],
					'unloadactionzoneid'=>$zrowcg["unloadactionzoneid"],
					'attachactionzoneid'=>$zrowcg["attachactionzoneid"],
					'alttag'=>$wtwconnect->escapeHTML($zrowcg["alttag"]),
					'createdate'=>$zrowcg["createdate"],
					'createuserid'=>$zrowcg["createuserid"],
					'updatedate'=>$zrowcg["updatedate"],
					'updateuserid'=>$zrowcg["updateuserid"]
				);
				$zcg += 1;
				addUserID($zrowcg["createuserid"]);
				addUserID($zrowcg["updateuserid"]);
			}

			/* get content ratings */
			$zresultscr = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."contentratings
				where webid='".$zwebid."'
					and webtype='".$zwebtype."'
					and deleted=0;");
			$zcr = 0;
			$zcontentratings = array();
			foreach ($zresultscr as $zrowcr) {
				$zcontentratings[$zcr] = array(
					'contentratingid'=>$zrowcr["contentratingid"],
					'pastcontentratingid'=>$zrowcr["pastcontentratingid"],
					'webid'=>$zrowcr["webid"],
					'webtype'=>$zrowcr["webtype"],
					'rating'=>$zrowcr["rating"],
					'ratingvalue'=>$zrowcr["ratingvalue"],
					'contentwarning'=>$zrowcr["contentwarning"],
					'createdate'=>$zrowcr["createdate"],
					'createuserid'=>$zrowcr["createuserid"],
					'updatedate'=>$zrowcr["updatedate"],
					'updateuserid'=>$zrowcr["updateuserid"]
				);
				$zcr += 1;
				addUserID($zrowcr["createuserid"]);
				addUserID($zrowcr["updateuserid"]);
			}

			/* get plugins required */
			$zresultplugins = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."pluginsrequired
				where webid='".$zwebid."'
					and webtype='".$zwebtype."'
					and deleted=0;");
			$zp = 0;
			$zpluginsrequired = array();
			foreach ($zresultplugins as $zrowp) {
				$zpluginsrequired[$zp] = array(
					'pluginsrequiredid'=>$zrowp["pluginsrequiredid"],
					'pastpluginsrequiredid'=>$zrowp["pastpluginsrequiredid"],
					'webid'=>$zrowp["webid"],
					'webtype'=>$zrowp["webtype"],
					'pluginname'=>$zrowp["pluginname"],
					'optional'=>$zrowp["optional"],
					'createdate'=>$zrowp["createdate"],
					'createuserid'=>$zrowp["createuserid"],
					'updatedate'=>$zrowp["updatedate"],
					'updateuserid'=>$zrowp["updateuserid"]
				);
				$zp += 1;
				addUserID($zrowp["createuserid"]);
				addUserID($zrowp["updateuserid"]);
			}

			/* get action zones */
			$zresultsaz = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."actionzones
				where ".$zwebtype."id='".$zwebid."'
					and deleted=0;");
			$zaz = 0;
			$zactionzones = array();
			foreach ($zresultsaz as $zrowaz) {

				/* get action zone animations */
				$zresultsazanim = $wtwconnect->query("
					select *
					from ".wtw_tableprefix."actionzoneanimations
					where actionzoneid='".$zrowaz["actionzoneid"]."'
						and deleted=0;");
				$zazanim = 0;
				$zazanimations = array();
				foreach ($zresultsazanim as $zrowsazanim) {
					$zazanimations[$zazanim] = array(
						'actionzoneanimationid'=>$zrowsazanim["actionzoneanimationid"],
						'actionzoneid'=>$zrowsazanim["actionzoneid"],
						'avataranimationid'=>$zrowsazanim["avataranimationid"],
						'createdate'=>$zrowsazanim["createdate"],
						'createuserid'=>$zrowsazanim["createuserid"],
						'updatedate'=>$zrowsazanim["updatedate"],
						'updateuserid'=>$zrowsazanim["updateuserid"]
					);
					$zazanim += 1;
					addAvatarAnimationID($zrowsazanim["avataranimationid"]);				
					addUserID($zrowsazanim["createuserid"]);
					addUserID($zrowsazanim["updateuserid"]);
				}
				
				/* get scripts */			
				$zresultsscripts = $wtwconnect->query("
					select *
					from ".wtw_tableprefix."scripts
					where webid='".$zwebid."'
						and actionzoneid='".$zrowaz["actionzoneid"]."'
						and deleted=0;");
				foreach ($zresultsscripts as $zrowscripts) {
					addScriptID($zrowscripts["scriptid"], $wtwconnect->domainurl."/content/uploads/".$zwebtypes."/".$zwebid."/");
				}
				
				/* add action zone to action zones array */
				$zactionzones[$zaz] = array(
					'actionzoneid'=>$zrowaz["actionzoneid"],
					'pastactionzoneid'=>$zrowaz["pastactionzoneid"],
					'communityid'=>$zrowaz["communityid"],
					'buildingid'=>$zrowaz["buildingid"],
					'thingid'=>$zrowaz["thingid"],
					'attachmoldid'=>$zrowaz["attachmoldid"],
					'loadactionzoneid'=>$zrowaz["loadactionzoneid"],
					'parentactionzoneid'=>$zrowaz["parentactionzoneid"],
					'teleportwebid'=>$zrowaz["teleportwebid"],
					'teleportwebtype'=>$zrowaz["teleportwebtype"],
					'spawnactionzoneid'=>$zrowaz["spawnactionzoneid"],
					'actionzonename'=>$wtwconnect->escapeHTML($zrowaz["actionzonename"]),
					'actionzonetype'=>$zrowaz["actionzonetype"],
					'actionzoneshape'=>$zrowaz["actionzoneshape"],
					'movementtype'=>$zrowaz["movementtype"],
					'movementdistance'=>$zrowaz["movementdistance"],
					'positionx'=>$zrowaz["positionx"],
					'positiony'=>$zrowaz["positiony"],
					'positionz'=>$zrowaz["positionz"],
					'scalingx'=>$zrowaz["scalingx"],
					'scalingy'=>$zrowaz["scalingy"],
					'scalingz'=>$zrowaz["scalingz"],
					'rotationx'=>$zrowaz["rotationx"],
					'rotationy'=>$zrowaz["rotationy"],
					'rotationz'=>$zrowaz["rotationz"],
					'axispositionx'=>$zrowaz["axispositionx"],
					'axispositiony'=>$zrowaz["axispositiony"],
					'axispositionz'=>$zrowaz["axispositionz"],
					'axisrotationx'=>$zrowaz["axisrotationx"],
					'axisrotationy'=>$zrowaz["axisrotationy"],
					'axisrotationz'=>$zrowaz["axisrotationz"],
					'rotateaxis'=>$zrowaz["rotateaxis"],
					'rotatedegrees'=>$zrowaz["rotatedegrees"],
					'rotatedirection'=>$zrowaz["rotatedirection"],
					'rotatespeed'=>$zrowaz["rotatespeed"],
					'value1'=>$zrowaz["value1"],
					'value2'=>$zrowaz["value2"],
					'defaulteditform'=>$zrowaz["defaulteditform"],
					'jsfunction'=>$zrowaz["jsfunction"],
					'jsparameters'=>$zrowaz["jsparameters"],
					'animations'=>$zazanimations,
					'createdate'=>$zrowaz["createdate"],
					'createuserid'=>$zrowaz["createuserid"],
					'updatedate'=>$zrowaz["updatedate"],
					'updateuserid'=>$zrowaz["updateuserid"]
				);
				$zaz += 1;
				addUserID($zrowaz["createuserid"]);
				addUserID($zrowaz["updateuserid"]);
			}

			/* get molds */
			$zresultsmolds = $wtwconnect->query("
				select *
				from ".wtw_tableprefix.$zwebtype."molds
				where ".$zwebtype."id='".$zwebid."'
					and deleted=0;");
			$zmold = 0;
			$zmolds = array();
			foreach ($zresultsmolds as $zrowmolds) {
				
				/* get mold points */
				$zresultsmp = $wtwconnect->query("
					select *
					from ".wtw_tableprefix."moldpoints
					where moldid='".$zrowmolds[$zwebtype."moldid"]."'
						and deleted=0;");
				$zmp = 0;
				$zmoldpoints = array();
				foreach ($zresultsmp as $zrowmp) {
					$zmoldpoints[$zmp] = array(
						'moldpointid'=>$zrowmp["moldpointid"],
						'pastmoldpointid'=>$zrowmp["pastmoldpointid"],
						'moldid'=>$zrowmp["moldid"],
						'pathnumber'=>$zrowmp["pathnumber"],
						'sorder'=>$zrowmp["sorder"],
						'positionx'=>$zrowmp["positionx"],
						'positiony'=>$zrowmp["positiony"],
						'positionz'=>$zrowmp["positionz"],
						'createdate'=>$zrowmp["createdate"],
						'createuserid'=>$zrowmp["createuserid"],
						'updatedate'=>$zrowmp["updatedate"],
						'updateuserid'=>$zrowmp["updateuserid"]
					);
					$zmp += 1;
					addUserID($zrowmp["createuserid"]);
					addUserID($zrowmp["updateuserid"]);
				}			

				/* get webimages */
				$zresultswi = $wtwconnect->query("
					select *
					from ".wtw_tableprefix."webimages
					where ".$zwebtype."moldid='".$zrowmolds[$zwebtype."moldid"]."'
						and deleted=0;");
				$zwi = 0;
				$zwebimages = array();
				foreach ($zresultswi as $zrowwi) {
					$zwebimages[$zwi] = array(
						'webimageid'=>$zrowwi["webimageid"],
						'pastwebimageid'=>$zrowwi["pastwebimageid"],
						'communitymoldid'=>$zrowwi["communitymoldid"],
						'buildingmoldid'=>$zrowwi["buildingmoldid"],
						'thingmoldid'=>$zrowwi["thingmoldid"],
						'imageindex'=>$zrowwi["imageindex"],
						'imageid'=>$zrowwi["imageid"],
						'imagehoverid'=>$zrowwi["imagehoverid"],
						'imageclickid'=>$zrowwi["imageclickid"],
						'graphiclevel'=>$zrowwi["graphiclevel"],
						'jsfunction'=>$zrowwi["jsfunction"],
						'jsparameters'=>$zrowwi["jsparameters"],
						'userid'=>$zrowwi["userid"],
						'alttag'=>$wtwconnect->escapeHTML($zrowwi["alttag"]),
						'createdate'=>$zrowwi["createdate"],
						'createuserid'=>$zrowwi["createuserid"],
						'updatedate'=>$zrowwi["updatedate"],
						'updateuserid'=>$zrowwi["updateuserid"]
					);
					$zwi += 1;
					addUploadID($zrowwi["imageid"], true);
					addUploadID($zrowwi["imagehoverid"], true);
					addUploadID($zrowwi["imageclickid"], true);
					addUserID($zrowwi["userid"]);
					addUserID($zrowwi["createuserid"]);
					addUserID($zrowwi["updateuserid"]);
				}			
				
				/* add molds to array */
				$zmolds[$zmold] = array(
					$zwebtype.'moldid'=>$zrowmolds[$zwebtype."moldid"],
					'past'.$zwebtype.'moldid'=>$zrowmolds["past".$zwebtype."moldid"],
					$zwebtype.'id'=>$zrowmolds[$zwebtype."id"],
					'loadactionzoneid'=>$zrowmolds["loadactionzoneid"],
					'unloadactionzoneid'=>$zrowmolds["unloadactionzoneid"],
					'shape'=>$zrowmolds["shape"],
					'covering'=>$zrowmolds["covering"],
					'positionx'=>$zrowmolds["positionx"],
					'positiony'=>$zrowmolds["positiony"],
					'positionz'=>$zrowmolds["positionz"],
					'scalingx'=>$zrowmolds["scalingx"],
					'scalingy'=>$zrowmolds["scalingy"],
					'scalingz'=>$zrowmolds["scalingz"],
					'rotationx'=>$zrowmolds["rotationx"],
					'rotationy'=>$zrowmolds["rotationy"],
					'rotationz'=>$zrowmolds["rotationz"],
					'special1'=>$zrowmolds["special1"],
					'special2'=>$zrowmolds["special2"],
					'uoffset'=>$zrowmolds["uoffset"],
					'voffset'=>$zrowmolds["voffset"],
					'uscale'=>$zrowmolds["uscale"],
					'vscale'=>$zrowmolds["vscale"],
					'uploadobjectid'=>$zrowmolds["uploadobjectid"],
					'graphiclevel'=>$zrowmolds["graphiclevel"],
					'textureid'=>$zrowmolds["textureid"],
					'texturebumpid'=>$zrowmolds["texturebumpid"],
					'texturehoverid'=>$zrowmolds["texturehoverid"],
					'videoid'=>$zrowmolds["videoid"],
					'videoposterid'=>$zrowmolds["videoposterid"],
					'diffusecolor'=> $zrowmolds["diffusecolor"],
					'emissivecolor'=> $zrowmolds["emissivecolor"],
					'specularcolor'=> $zrowmolds["specularcolor"],
					'ambientcolor'=> $zrowmolds["ambientcolor"],
					'heightmapid'=>$zrowmolds["heightmapid"],
					'mixmapid'=>$zrowmolds["mixmapid"],
					'texturerid'=>$zrowmolds["texturerid"],
					'texturegid'=>$zrowmolds["texturegid"],
					'texturebid'=>$zrowmolds["texturebid"],
					'texturebumprid'=>$zrowmolds["texturebumprid"],
					'texturebumpgid'=>$zrowmolds["texturebumpgid"],
					'texturebumpbid'=>$zrowmolds["texturebumpbid"],
					'soundid'=>$zrowmolds["soundid"],
					'soundname'=>$zrowmolds["soundname"],
					'soundloop'=>$zrowmolds["soundloop"],
					'soundmaxdistance'=>$zrowmolds["soundmaxdistance"],
					'soundrollofffactor'=>$zrowmolds["soundrollofffactor"],
					'soundrefdistance'=>$zrowmolds["soundrefdistance"],
					'soundconeinnerangle'=>$zrowmolds["soundconeinnerangle"],
					'soundconeouterangle'=>$zrowmolds["soundconeouterangle"],
					'soundconeoutergain'=>$zrowmolds["soundconeoutergain"],
					'webtext'=>$wtwconnect->escapeHTML($zrowmolds["webtext"]),
					'webstyle'=>$wtwconnect->escapeHTML($zrowmolds["webstyle"]),
					'opacity'=>$zrowmolds["opacity"],
					'sideorientation'=>$zrowmolds["sideorientation"],
					'billboard'=>$zrowmolds["billboard"],
					'waterreflection'=>$zrowmolds["waterreflection"],
					'receiveshadows'=>$zrowmolds["receiveshadows"],
					'castshadows'=>$zrowmolds["castshadows"],
					'subdivisions'=>$zrowmolds["subdivisions"],
					'minheight'=>$zrowmolds["minheight"],
					'maxheight'=>$zrowmolds["maxheight"],
					'checkcollisions'=>$zrowmolds["checkcollisions"],
					'ispickable'=>$zrowmolds["ispickable"],
					'actionzoneid'=>$zrowmolds["actionzoneid"],
					'actionzone2id'=>$zrowmolds["actionzone2id"],
					'csgmoldid'=>$zrowmolds["csgmoldid"],
					'csgaction'=>$zrowmolds["csgaction"],
					'physicsenabled'=>$zrowmolds["physicsenabled"],
					'physicscenterx'=>$zrowmolds["physicscenterx"],
					'physicscentery'=>$zrowmolds["physicscentery"],
					'physicscenterz'=>$zrowmolds["physicscenterz"],
					'physicsextentsx'=>$zrowmolds["physicsextentsx"],
					'physicsextentsy'=>$zrowmolds["physicsextentsy"],
					'physicsextentsz'=>$zrowmolds["physicsextentsz"],
					'physicsfriction'=>$zrowmolds["physicsfriction"],
					'physicsistriggershape'=>$zrowmolds["physicsistriggershape"],
					'physicsmass'=>$zrowmolds["physicsmass"],
					'physicspointax'=>$zrowmolds["physicspointax"],
					'physicspointay'=>$zrowmolds["physicspointay"],
					'physicspointaz'=>$zrowmolds["physicspointaz"],
					'physicspointbx'=>$zrowmolds["physicspointbx"],
					'physicspointby'=>$zrowmolds["physicspointby"],
					'physicspointbz'=>$zrowmolds["physicspointbz"],
					'physicsradius'=>$zrowmolds["physicsradius"],
					'physicsrestitution'=>$zrowmolds["physicsrestitution"],
					'physicsrotationx'=>$zrowmolds["physicsrotationx"],
					'physicsrotationy'=>$zrowmolds["physicsrotationy"],
					'physicsrotationz'=>$zrowmolds["physicsrotationz"],
					'physicsrotationw'=>$zrowmolds["physicsrotationw"],
					'physicsstartasleep'=>$zrowmolds["physicsstartasleep"],
					'alttag'=>$wtwconnect->escapeHTML($zrowmolds["alttag"]),
					'jsfunction'=>$zrowmolds["jsfunction"],
					'jsparameters'=>$zrowmolds["jsparameters"],
					'createdate'=>$zrowmolds["createdate"],
					'createuserid'=>$zrowmolds["createuserid"],
					'updatedate'=>$zrowmolds["updatedate"],
					'updateuserid'=>$zrowmolds["updateuserid"],
					'moldpoints'=>$zmoldpoints,
					'webimages'=>$zwebimages
				);
				$zmold += 1;
				addUploadObjectID($zrowmolds["uploadobjectid"]);
				addUploadID($zrowmolds["textureid"], true);
				addUploadID($zrowmolds["texturebumpid"], true);
				addUploadID($zrowmolds["texturehoverid"], true);
				addUploadID($zrowmolds["videoid"], false);
				addUploadID($zrowmolds["videoposterid"], true);
				addUploadID($zrowmolds["heightmapid"], true);
				addUploadID($zrowmolds["mixmapid"], true);
				addUploadID($zrowmolds["texturerid"], true);
				addUploadID($zrowmolds["texturegid"], true);
				addUploadID($zrowmolds["texturebid"], true);
				addUploadID($zrowmolds["texturebumprid"], true);
				addUploadID($zrowmolds["texturebumpgid"], true);
				addUploadID($zrowmolds["texturebumpbid"], true);
				addUploadID($zrowmolds["soundid"], false);
				addUserID($zrowmolds["createuserid"]);
				addUserID($zrowmolds["updateuserid"]);
			}

			/* json structured response */
			addUploadID($zrow["snapshotid"], true);
			addUserID($zrow["userid"]);
			addUserID($zrow["shareuserid"]);
			addUserID($zrow["createuserid"]);
			addUserID($zrow["updateuserid"]);

			$ztextureid = '';
			$zskydomeid = '';
			$zgroundpositiony = '';
			$zwaterpositiony = '';
			$zwaterbumpheight = .6;
			$zwatersubdivisions = 2;
			$zwindforce = -10;
			$zwinddirectionx = 1;
			$zwinddirectiony = 0;
			$zwinddirectionz = 1;
			$zwaterwaveheight = .2;
			$zwaterwavelength = .02;
			$zwatercolorrefraction = '#23749C';
			$zwatercolorreflection = '#52BCF1';
			$zwatercolorblendfactor = .2;
			$zwatercolorblendfactor2 = .2;
			$zwateralpha = .9;
			$zwaterbumpid = '';
			$zsceneambientcolor = '#E5E8E8';
			$zsceneclearcolor = '#000000';
			$zsceneuseclonedmeshmap = 1;
			$zsceneblockmaterialdirtymechanism = 1;
			$zscenefogenabled = 0;
			$zscenefogmode = '';
			$zscenefogdensity = .01;
			$zscenefogstart = 20;
			$zscenefogend = 60;
			$zscenefogcolor = '#c0c0c0';
			$zsundirectionalintensity = 1;
			$zsundiffusecolor = '#ffffff';
			$zsunspecularcolor = '#ffffff';
			$zsungroundcolor = '#000000';
			$zsundirectionx = 999;
			$zsundirectiony = -999;
			$zsundirectionz = 999;
			$zsunpositionx = 0;
			$zsunpositiony = 1000;
			$zsunpositionz = 0;
			$zbacklightintensity = 0.5;
			$zbacklightdirectionx = -999;
			$zbacklightdirectiony = 999;
			$zbacklightdirectionz = -999;
			$zbacklightpositionx = 0;
			$zbacklightpositiony = 1000;
			$zbacklightpositionz = 0;
			$zbacklightdiffusecolor = '#ffffff';
			$zbacklightspecularcolor = '#ffffff';
			$zskytype = '';
			$zskysize = 5000;
			$zskyboxfolder = '';
			$zskyboxfile = '';
			$zskyboximageleft = '';
			$zskyboximageup = '';
			$zskyboximagefront = '';
			$zskyboximageright = '';
			$zskyboximagedown = '';
			$zskyboximageback = '';
			$zskypositionoffsetx = 0;
			$zskypositionoffsety = 0;
			$zskypositionoffsetz = 0;
			$zskyboxmicrosurface = 0;
			$zskyboxpbr = 0;
			$zskyboxasenvironmenttexture = 0;
			$zskyboxblur = 0;
			$zskyboxdiffusecolor = '#000000';
			$zskyboxspecularcolor = '#000000';
			$zskyboxambientcolor = '#000000';
			$zskyboxemissivecolor = '#000000';
			$zskyinclination = 0;
			$zskyluminance = 1;
			$zskyazimuth = 0.25;
			$zskyrayleigh = 2;
			$zskyturbidity = 10;
			$zskymiedirectionalg = .8;
			$zskymiecoefficient = .008;
			$zbuildingpositionx = 0;
			$zbuildingpositiony = 0;
			$zbuildingpositionz = 0;
			$zbuildingscalingx = 1;
			$zbuildingscalingy = 1;
			$zbuildingscalingz = 1;
			$zbuildingrotationx = 0;
			$zbuildingrotationy = 0;
			$zbuildingrotationz = 0;
			
			if ($zwebtype == 'community') {
				/* these fields only apply to 3D Community Scenes */
				$ztextureid = $zrow["textureid"];
				$zskydomeid = $zrow["skydomeid"];
				$zgroundpositiony = $zrow["groundpositiony"];
				$zwaterpositiony = $zrow["waterpositiony"];
				if (isset($zrow["waterbumpheight"])) {
					$zwaterbumpheight = $zrow["waterbumpheight"];
				}
				if (isset($zrow["watersubdivisions"])) {
					$zwatersubdivisions = $zrow["watersubdivisions"];
				}
				if (isset($zrow["windforce"])) {
					$zwindforce = $zrow["windforce"];
				}
				if (isset($zrow["winddirectionx"])) {
					$zwinddirectionx = $zrow["winddirectionx"];
				}
				if (isset($zrow["winddirectiony"])) {
					$zwinddirectiony = $zrow["winddirectiony"];
				}
				if (isset($zrow["winddirectionz"])) {
					$zwinddirectionz = $zrow["winddirectionz"];
				}
				if (isset($zrow["waterwaveheight"])) {
					$zwaterwaveheight = $zrow["waterwaveheight"];
				}
				if (isset($zrow["waterwavelength"])) {
					$zwaterwavelength = $zrow["waterwavelength"];
				}
				$zwatercolorrefraction = $zrow["watercolorrefraction"];
				$zwatercolorreflection = $zrow["watercolorreflection"];
				if (isset($zrow["watercolorblendfactor"])) {
					$zwatercolorblendfactor = $zrow["watercolorblendfactor"];
				}
				if (isset($zrow["watercolorblendfactor2"])) {
					$zwatercolorblendfactor2 = $zrow["watercolorblendfactor2"];
				}
				if (isset($zrow["wateralpha"])) {
					$zwateralpha = $zrow["wateralpha"];
				}
				$zwaterbumpid = $zrow["waterbumpid"];

				$zsceneambientcolor = $zrow["sceneambientcolor"];
				$zsceneclearcolor = $zrow["sceneclearcolor"];
				if (isset($zrow["sceneuseclonedmeshmap"])) {
					$zsceneuseclonedmeshmap = $zrow["sceneuseclonedmeshmap"];
				}
				if (isset($zrow["sceneblockmaterialdirtymechanism"])) {
					$zsceneblockmaterialdirtymechanism = $zrow["sceneblockmaterialdirtymechanism"];
				}
				if (isset($zrow["sundirectionalintensity"])) {
					$zsundirectionalintensity = $zrow["sundirectionalintensity"];
				}
				$zsundiffusecolor = $zrow["sundiffusecolor"];
				$zsunspecularcolor = $zrow["sunspecularcolor"];
				$zsungroundcolor = $zrow["sungroundcolor"];
				if (isset($zrow["sundirectionx"])) {
					$zsundirectionx = $zrow["sundirectionx"];
				}
				if (isset($zrow["sundirectiony"])) {
					$zsundirectiony = $zrow["sundirectiony"];
				}
				if (isset($zrow["sundirectionz"])) {
					$zsundirectionz = $zrow["sundirectionz"];
				}
				if (isset($zrow["sunpositionx"])) {
					$zsunpositionx = $zrow["sunpositionx"];
				}
				if (isset($zrow["sunpositiony"])) {
					$zsunpositiony = $zrow["sunpositiony"];
				}
				if (isset($zrow["sunpositionz"])) {
					$zsunpositionz = $zrow["sunpositionz"];
				}
				if (isset($zrow["backlightintensity"])) {
					$zbacklightintensity = $zrow["backlightintensity"];
				}
				if (isset($zrow["backlightdirectionx"])) {
					$zbacklightdirectionx = $zrow["backlightdirectionx"];
				}
				if (isset($zrow["backlightdirectiony"])) {
					$zbacklightdirectiony = $zrow["backlightdirectiony"];
				}
				if (isset($zrow["backlightdirectionz"])) {
					$zbacklightdirectionz = $zrow["backlightdirectionz"];
				}
				if (isset($zrow["backlightpositionx"])) {
					$zbacklightpositionx = $zrow["backlightpositionx"];
				}
				if (isset($zrow["backlightpositiony"])) {
					$zbacklightpositiony = $zrow["backlightpositiony"];
				}
				if (isset($zrow["backlightpositionz"])) {
					$zbacklightpositionz = $zrow["backlightpositionz"];
				}
				$zbacklightdiffusecolor = $zrow["backlightdiffusecolor"];
				$zbacklightspecularcolor = $zrow["backlightspecularcolor"];
				$zscenefogenabled = $zrow["scenefogenabled"];
				$zscenefogmode = $zrow["scenefogmode"];
				$zscenefogdensity = $zrow["scenefogdensity"];
				$zscenefogstart = $zrow["scenefogstart"];
				$zscenefogend = $zrow["scenefogend"];
				$zscenefogcolor = $zrow["scenefogcolor"];
				$zskytype = $zrow["skytype"];
				if (isset($zrow["skysize"])) {
					$zskysize = $zrow["skysize"];
				}
				$zskyboxfolder = $zrow["skyboxfolder"];
				$zskyboxfile = $zrow["skyboxfile"];
				$zskyboximageleft = $zrow["skyboximageleft"];
				$zskyboximageup = $zrow["skyboximageup"];
				$zskyboximagefront = $zrow["skyboximagefront"];
				$zskyboximageright = $zrow["skyboximageright"];
				$zskyboximagedown = $zrow["skyboximagedown"];
				$zskyboximageback = $zrow["skyboximageback"];
				if (isset($zrow["skypositionoffsetx"])) {
					$zskypositionoffsetx = $zrow["skypositionoffsetx"];
				}
				if (isset($zrow["skypositionoffsety"])) {
					$zskypositionoffsety = $zrow["skypositionoffsety"];
				}
				if (isset($zrow["skypositionoffsetz"])) {
					$zskypositionoffsetz = $zrow["skypositionoffsetz"];
				}
				if (isset($zrow["skyboxmicrosurface"])) {
					$zskyboxmicrosurface = $zrow["skyboxmicrosurface"];
				}
				if (isset($zrow["skyboxpbr"])) {
					$zskyboxpbr = $zrow["skyboxpbr"];
				}
				if (isset($zrow["skyboxasenvironmenttexture"])) {
					$zskyboxasenvironmenttexture = $zrow["skyboxasenvironmenttexture"];
				}
				if (isset($zrow["skyboxblur"])) {
					$zskyboxblur = $zrow["skyboxblur"];
				}
				$zskyboxdiffusecolor = $zrow["skyboxdiffusecolor"];
				$zskyboxspecularcolor = $zrow["skyboxspecularcolor"];
				$zskyboxambientcolor = $zrow["skyboxambientcolor"];
				$zskyboxemissivecolor = $zrow["skyboxemissivecolor"];
				if (isset($zrow["skyinclination"])) {
					$zskyinclination = $zrow["skyinclination"];
				}
				if (isset($zrow["skyluminance"])) {
					$zskyluminance = $zrow["skyluminance"];
				}
				if (isset($zrow["skyazimuth"])) {
					$zskyazimuth = $zrow["skyazimuth"];
				}
				if (isset($zrow["skyrayleigh"])) {
					$zskyrayleigh = $zrow["skyrayleigh"];
				}
				if (isset($zrow["skyturbidity"])) {
					$zskyturbidity = $zrow["skyturbidity"];
				}
				if (isset($zrow["skymiedirectionalg"])) {
					$zskymiedirectionalg = $zrow["skymiedirectionalg"];
				}
				if (isset($zrow["skymiecoefficient"])) {
					$zskymiecoefficient = $zrow["skymiecoefficient"];
				}
				if (isset($zrow["buildingpositionx"])) {
					$zbuildingpositionx = $zrow["buildingpositionx"];
				}
				if (isset($zrow["buildingpositiony"])) {
					$zbuildingpositiony = $zrow["buildingpositiony"];
				}
				if (isset($zrow["buildingpositionz"])) {
					$zbuildingpositionz = $zrow["buildingpositionz"];
				}
				if (isset($zrow["buildingscalingx"])) {
					$zbuildingscalingx = $zrow["buildingscalingx"];
				}
				if (isset($zrow["buildingscalingy"])) {
					$zbuildingscalingy = $zrow["buildingscalingy"];
				}
				if (isset($zrow["buildingscalingz"])) {
					$zbuildingscalingz = $zrow["buildingscalingz"];
				}
				if (isset($zrow["buildingrotationx"])) {
					$zbuildingrotationx = $zrow["buildingrotationx"];
				}
				if (isset($zrow["buildingrotationy"])) {
					$zbuildingrotationy = $zrow["buildingrotationy"];
				}
				if (isset($zrow["buildingrotationz"])) {
					$zbuildingrotationz = $zrow["buildingrotationz"];
				}

				addUploadID($ztextureid, true);
				addUploadID($zskydomeid, true);
			}

			$zresponse = array(
				'serverinstanceid' => wtw_serverinstanceid,
				'domainurl' => $wtwconnect->domainurl,
				$zwebtype.'id' => $zrow[$zwebtype."id"],
				'past'.$zwebtype.'id' => $zrow["past".$zwebtype."id"],
				$zwebtype.'name' => $wtwconnect->escapeHTML($zrow[$zwebtype."name"]),
				$zwebtype.'description' => $wtwconnect->escapeHTML($zrow[$zwebtype."description"]),
				'analyticsid' => '',
				'versionid' => $zrow["versionid"],
				'version' => $zrow["version"],
				'versionorder' => $zrow["versionorder"],
				'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
				'userid' => $zrow["userid"],
				'positionx' => $zrow["positionx"],
				'positiony' => $zrow["positiony"],
				'positionz' => $zrow["positionz"],
				'scalingx' => $zrow["scalingx"],
				'scalingy' => $zrow["scalingy"],
				'scalingz' => $zrow["scalingz"],
				'rotationx' => $zrow["rotationx"],
				'rotationy' => $zrow["rotationy"],
				'rotationz' => $zrow["rotationz"],
				'textureid' => $ztextureid,
				'skydomeid' => $zskydomeid,
				'gravity' => $zrow["gravity"],
				'groundpositiony' => $zgroundpositiony,
				'waterpositiony' => $zwaterpositiony,
				'waterbumpheight' => $zwaterbumpheight,
				'watersubdivisions' => $zwatersubdivisions,
				'windforce' => $zwindforce,
				'winddirectionx' => $zwinddirectionx,
				'winddirectiony' => $zwinddirectiony,
				'winddirectionz' => $zwinddirectionz,
				'waterwaveheight' => $zwaterwaveheight,
				'waterwavelength' => $zwaterwavelength,
				'watercolorrefraction' => $zwatercolorrefraction,
				'watercolorreflection' => $zwatercolorreflection,
				'watercolorblendfactor' => $zwatercolorblendfactor,
				'watercolorblendfactor2' => $zwatercolorblendfactor2,
				'wateralpha' => $zwateralpha,
				'waterbumpid' => $zwaterbumpid,
				'sceneambientcolor' => $zsceneambientcolor,
				'sceneclearcolor' => $zsceneclearcolor,
				'sceneuseclonedmeshmap' => $zsceneuseclonedmeshmap,
				'sceneblockmaterialdirtymechanism' => $zsceneblockmaterialdirtymechanism,
				'sundirectionalintensity' => $zsundirectionalintensity,
				'sundiffusecolor' => $zsundiffusecolor,
				'sunspecularcolor' => $zsunspecularcolor,
				'sungroundcolor' => $zsungroundcolor,
				'sundirectionx' => $zsundirectionx,
				'sundirectiony' => $zsundirectiony,
				'sundirectionz' => $zsundirectionz,
				'sunpositionx' => $zsunpositionx,
				'sunpositiony' => $zsunpositiony,
				'sunpositionz' => $zsunpositionz,
				'backlightintensity' => $zbacklightintensity,
				'backlightdirectionx' => $zbacklightdirectionx,
				'backlightdirectiony' => $zbacklightdirectiony,
				'backlightdirectionz' => $zbacklightdirectionz,
				'backlightpositionx' => $zbacklightpositionx,
				'backlightpositiony' => $zbacklightpositiony,
				'backlightpositionz' => $zbacklightpositionz,
				'backlightdiffusecolor' => $zbacklightdiffusecolor,
				'backlightspecularcolor' => $zbacklightspecularcolor,
				'scenefogenabled' => $zscenefogenabled,
				'scenefogmode' => $zscenefogmode,
				'scenefogdensity' => $zscenefogdensity,
				'scenefogstart' => $zscenefogstart,
				'scenefogend' => $zscenefogend,
				'scenefogcolor' => $zscenefogcolor,
				'skytype' => $zskytype,
				'skysize' => $zskysize,
				'skyboxfolder' => $zskyboxfolder,
				'skyboxfile' => $zskyboxfile,
				'skyboximageleft' => $zskyboximageleft,
				'skyboximageup' => $zskyboximageup,
				'skyboximagefront'=> $zskyboximagefront,
				'skyboximageright' => $zskyboximageright,
				'skyboximagedown' => $zskyboximagedown,
				'skyboximageback' => $zskyboximageback,
				'skypositionoffsetx' => $zskypositionoffsetx,
				'skypositionoffsety' => $zskypositionoffsety,
				'skypositionoffsetz' => $zskypositionoffsetz,
				'skyboxmicrosurface' => $zskyboxmicrosurface,
				'skyboxpbr' => $zskyboxpbr,
				'skyboxasenvironmenttexture' => $zskyboxasenvironmenttexture,
				'skyboxblur' => $zskyboxblur,
				'skyboxdiffusecolor' => $zskyboxdiffusecolor,
				'skyboxspecularcolor' => $zskyboxspecularcolor,
				'skyboxambientcolor' => $zskyboxambientcolor,
				'skyboxemissivecolor' => $zskyboxemissivecolor,
				'skyinclination' => $zskyinclination,
				'skyluminance' => $zskyluminance,
				'skyazimuth' => $zskyazimuth,
				'skyrayleigh' => $zskyrayleigh,
				'skyturbidity' => $zskyturbidity,
				'skymiedirectionalg' => $zskymiedirectionalg,
				'skymiecoefficient' => $zskymiecoefficient,
				'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
				'tags' => $wtwconnect->escapeHTML($zrow["tags"]),
				'description' => $wtwconnect->escapeHTML($zrow["description"]),
				'snapshotid' => $zrow["snapshotid"],
				'shareuserid' => $zrow["shareuserid"],
				'alttag' => $wtwconnect->escapeHTML($zrow["alttag"]),
				'buildingpositionx' => $zbuildingpositionx,
				'buildingpositiony' => $zbuildingpositiony,
				'buildingpositionz' => $zbuildingpositionz,
				'buildingscalingx' => $zbuildingscalingx,
				'buildingscalingy' => $zbuildingscalingy,
				'buildingscalingz' => $zbuildingscalingz,
				'buildingrotationx' => $zbuildingrotationx,
				'buildingrotationy' => $zbuildingrotationy,
				'buildingrotationz' => $zbuildingrotationz,
				'createdate' => $zrow["createdate"],
				'createuserid' => $zrow["createuserid"],
				'updatedate' => $zrow["updatedate"],
				'updateuserid' => $zrow["updateuserid"],
				'connectinggrids' => $zconnectinggrids,
				'childconnectinggrids' => $zchildconnectinggrids,
				'actionzones' => $zactionzones,
				'molds' => $zmolds,
				'uploadobjects' => $zuploadobjects,
				'uploads' => $zuploads,
				'scripts' => $zscripts,
				'contentratings' => $zcontentratings,
				'pluginsrequired' => $zpluginsrequired,
				'avataranimations' => $zavataranimations,
				'users' => $zusers
			);
		}
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-share.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: shareavatar.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides shared 3D Avatar information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;

$zuploads = array();
$zupload = 0;
$zavatarparts = array();
$zavataranimationdefs = array();


$zuploadobjects = array();
$zuploadobject = 0;
$zscripts = array();
$zscript = 0;
$zavataranimations = array();
$zavataranimation = 0;
$zusers = array();
$zuser = 0;

function addUploadID($zuploadid, $zrecursive) {
	global $wtwconnect;
	try {
		global $zuploads;
		global $zupload;
		if ($wtwconnect->hasValue($zuploadid)) {
			$zfound = false;
			foreach ($zuploads as $zrowup) {
				if ($zrowup["uploadid"] == $zuploadid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select upload file data */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."uploads
					where uploadid='".$zuploadid."'
					and deleted=0;");
				foreach ($zresults as $zrow) {
					$zfilepath = $zrow["filepath"];
					if ($wtwconnect->hasValue($zfilepath)) {
						if (substr($zfilepath, 0, 4) != "http") {
							$zfilepath = $wtwconnect->domainurl.$zfilepath;
						}
					}
					$zuploads[$zupload] = array (
						'uploadid'=>$zrow["uploadid"],
						'pastuploadid'=>$zrow["pastuploadid"],
						'originalid'=>$zrow["originalid"],
						'websizeid'=>$zrow["websizeid"],
						'thumbnailid'=>$zrow["thumbnailid"],
						'userid'=>$zrow["userid"],
						'filetitle'=>$wtwconnect->escapeHTML($zrow["filetitle"]),
						'filename'=>$zrow["filename"],
						'fileextension'=>$zrow["fileextension"],
						'filesize'=>$zrow["filesize"],
						'filetype'=>$zrow["filetype"],
						'filepath'=>$zfilepath,
						'filedata'=>$zrow["filedata"],
						'imagewidth'=>$zrow["imagewidth"],
						'imageheight'=>$zrow["imageheight"],
						'stock'=>$zrow["stock"],
						'hidedate'=>$zrow["hidedate"],
						'hideuserid'=>$zrow["hideuserid"],
						'hide'=>$zrow["hide"],
						'checkeddate'=>$zrow["checkeddate"],
						'checkeduserid'=>$zrow["checkeduserid"],
						'checked'=>$zrow["checked"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"]
					);
					$zupload += 1;
					addUserID($zrow["userid"]);
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-shareavatar.php-addUploadID=".$e->getMessage());
	}
}

function addAvatarAnimationID($zavataranimationid) {
	global $wtwconnect;
	try {
		global $zavataranimations;
		global $zavataranimation;
		if ($wtwconnect->hasValue($zavataranimationid)) {
			$zfound = false;
			foreach ($zavataranimations as $zrowanim) {
				if ($zrowanim["avataranimationid"] == $zavataranimationid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select avatar animations */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."avataranimations
					where avataranimationid='".$zavataranimationid."'
					limit 1;");
				foreach ($zresults as $zrow) {
					$zanimationicon = '';
					if (!empty($zrow["animationicon"])) {
						$zanimationicon = $wtwconnect->domainurl.$zrow["animationicon"];
					}
					
					$zavataranimations[$zavataranimation] = array (
						'avataranimationid'=>$zrow["avataranimationid"],
						'pastavataranimationid'=>$zrow["pastavataranimationid"],
						'avatarid'=>$zrow["avatarid"],
						'userid'=>$zrow["userid"],
						'loadpriority'=>$zrow["loadpriority"],
						'animationevent'=>$zrow["animationevent"],
						'animationfriendlyname'=>$zrow["animationfriendlyname"],
						'animationicon'=>$zanimationicon,
						'objectfolder'=>$wtwconnect->domainurl.$zrow["objectfolder"],
						'objectfile'=>$zrow["objectfile"],
						'startframe'=>$zrow["startframe"],
						'endframe'=>$zrow["endframe"],
						'animationloop'=>$zrow["animationloop"],
						'speedratio'=>$zrow["speedratio"],
						'soundid'=>$zrow["soundid"],
						'soundmaxdistance'=>$zrow["soundmaxdistance"],
						'createdate'=>$zrow["createdate"],
						'createuserid'=>$zrow["createuserid"],
						'updatedate'=>$zrow["updatedate"],
						'updateuserid'=>$zrow["updateuserid"]
					);
					$zavataranimation += 1;
					addUploadID($zrow["soundid"], false);
					addUserID($zrow["userid"]);
					addUserID($zrow["createuserid"]);
					addUserID($zrow["updateuserid"]);
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-shareavatar.php-addAvatarAnimationID=".$e->getMessage());
	}
}

function addUserID($zuserid) {
	global $wtwconnect;
	try {
		global $zusers;
		global $zuser;
		if ($wtwconnect->hasValue($zuserid)) {
			$zfound = false;
			foreach ($zusers as $zrowup) {
				if ($zrowup["userid"] == $zuserid) {
					$zfound = true;
				}
			}
			if (!$zfound) {
				/* select user */
				$zresults = $wtwconnect->query("
					select * 
					from ".wtw_tableprefix."users
					where userid='".$zuserid."'
					limit 1;");
				foreach ($zresults as $zrow) {
					$zusers[$zuser] = array (
						'userid'=>$zrow["userid"],
						'displayname'=>$wtwconnect->escapeHTML($zrow["displayname"]),
						'email'=>$zrow["email"],
						'uploadpathid'=>$zrow["uploadpathid"]
					);
					$zuser += 1;
				}
			}
		}
	} catch (Exception $e) {
		$wtwconnect->serror("connect-shareavatar.php-addUserID=".$e->getMessage());
	}
}

try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/shareavatar.php");
	
	/* get values from querystring or session */
	$zavatarid = $wtwconnect->getVal('avatarid','');
	$zuserid = $wtwconnect->getVal('userid','');
	$zsharehash = $wtwconnect->getVal('sharehash','');

	$zresponse = array();

	addUserID($zuserid);
	
//	echo $wtwconnect->addConnectHeader('*');
	header('Access-Control-Allow-Origin: *');
	header('Content-type: application/json');
	
	/* get 3D Avatar */
	$zresults = $wtwconnect->query("
		select *
		from ".wtw_tableprefix."avatars
		where avatarid='".$zavatarid."'
			and shareuserid='".$zuserid."'
			and sharehash='".$zsharehash."'
			and deleted=0
		limit 1;");

	/* format json return dataset */
	foreach ($zresults as $zrow) {
		/* get avatar parts from avatar colors table */
		$zresultsap = $wtwconnect->query("
			select *
			from ".wtw_tableprefix."avatarcolors
			where avatarid='".$zavatarid."'
				and deleted=0;");
		$zpart = 0;
		$zavatarparts = array();
		foreach ($zresultsap as $zrowap) {
			$zavatarparts[$zpart] = array(
				'avatarpartid'=>$zrowap["avatarpartid"],
				'pastavatarpartid'=>$zrowap["pastavatarpartid"],
				'avatarpart'=>$zrowap["avatarpart"],
				'diffusecolor'=>$zrowap["diffusecolor"],
				'specularcolor'=>$zrowap["specularcolor"],
				'emissivecolor'=>$zrowap["emissivecolor"],
				'ambientcolor'=>$zrowap["ambientcolor"],
				'createdate'=>$zrowap["createdate"],
				'createuserid'=>$zrowap["createuserid"],
				'updatedate'=>$zrowap["updatedate"],
				'updateuserid'=>$zrowap["updateuserid"]
			);
			$zpart += 1;
			addUserID($zrowap["createuserid"]);
			addUserID($zrowap["updateuserid"]);
		}

		/* get avatar animations */
		$zresultsa = $wtwconnect->query("
			select a1.* 
				from ".wtw_tableprefix."avataranimations a1
				inner join (
					select animationevent, max(updatedate) as updatedate, max(avataranimationid) as avataranimationid 
					from ".wtw_tableprefix."avataranimations 
					where avatarid='".$zavatarid."'
						and deleted=0
						and not animationevent='onoption'
					group by animationevent) a2
				on a1.avataranimationid = a2.avataranimationid
				where a1.avatarid='".$zavatarid."' 
					and a1.deleted=0
			union
			select a3.* 
				from ".wtw_tableprefix."avataranimations a3
				inner join (
					select animationfriendlyname, max(updatedate) as updatedate, max(avataranimationid) as avataranimationid 
					from ".wtw_tableprefix."avataranimations 
					where avatarid='".$zavatarid."'
						and deleted=0
						and animationevent='onoption'
					group by animationfriendlyname) a4
				on a3.avataranimationid = a4.avataranimationid
				where a3.avatarid='".$zavatarid."' 
					and a3.deleted=0
			order by loadpriority desc, animationevent, animationfriendlyname, avataranimationid;");
		$zanim = 0;
		foreach ($zresultsa as $zrowa) {
			if (!empty($zrowa["endframe"])) {
				$zavataranimationdefs[$zanim] = array(
					'avataranimationid'=>$zrowa["avataranimationid"],
					'pastavataranimationid'=>$zrowa["pastavataranimationid"],
					'loadpriority'=>$zrowa["loadpriority"],
					'animationevent'=>$zrowa["animationevent"],
					'animationfriendlyname'=>$zrowa["animationfriendlyname"],
					'animationicon'=>$zrowa["animationicon"],
					'objectfolder'=>$zrowa["objectfolder"],
					'objectfile'=>$zrowa["objectfile"],
					'startframe'=>$zrowa["startframe"],
					'endframe'=>$zrowa["endframe"],
					'animationloop'=>$zrowa["animationloop"],
					'speedratio'=>$zrowa["speedratio"],
					'soundid'=>$zrowa["soundid"],
					'soundmaxdistance'=>$zrowa["soundmaxdistance"],
					'createdate'=>$zrowa["createdate"],
					'createuserid'=>$zrowa["createuserid"],
					'updatedate'=>$zrowa["updatedate"],
					'updateuserid'=>$zrowa["updateuserid"]
				);
				$zanim += 1;
			}
			addUserID($zrowa["createuserid"]);
			addUserID($zrowa["updateuserid"]);
		}

		/* get content ratings */
		$zresultscr = $wtwconnect->query("
			select *
			from ".wtw_tableprefix."contentratings
			where webid='".$zavatarid."'
				and webtype='avatar'
				and deleted=0;");
		$zcr = 0;
		$zcontentratings = array();
		foreach ($zresultscr as $zrowcr) {
			$zcontentratings[$zcr] = array(
				'contentratingid'=>$zrowcr["contentratingid"],
				'pastcontentratingid'=>$zrowcr["pastcontentratingid"],
				'webid'=>$zrowcr["webid"],
				'webtype'=>$zrowcr["webtype"],
				'rating'=>$zrowcr["rating"],
				'ratingvalue'=>$zrowcr["ratingvalue"],
				'contentwarning'=>$zrowcr["contentwarning"],
				'createdate'=>$zrowcr["createdate"],
				'createuserid'=>$zrowcr["createuserid"],
				'updatedate'=>$zrowcr["updatedate"],
				'updateuserid'=>$zrowcr["updateuserid"]
			);
			$zcr += 1;
			addUserID($zrowcr["createuserid"]);
			addUserID($zrowcr["updateuserid"]);
		}

		/* json structured response */
		addUploadID($zrow["snapshotid"], true);
		addUserID($zrow["shareuserid"]);
		addUserID($zrow["createuserid"]);
		addUserID($zrow["updateuserid"]);
		
		$zfiles = array();
		
		$i = 0;
	
		$zfiles = $wtwconnect->getAvatarFilesList($zfiles, wtw_rootpath.$zrow["objectfolder"]);
		
		
		$zresponse = array(
			'serverinstanceid'=>wtw_serverinstanceid,
			'domainurl'=>$wtwconnect->domainurl,
			'avatarid' => $zrow["avatarid"],
			'pastavatarid' => $zrow["pastavatarid"],
			'versionid'=> $zrow["versionid"],
			'version'=> $zrow["version"],
			'versionorder'=> $zrow["versionorder"],
			'versiondesc'=> $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'avatargroup' => $zrow["avatargroup"],
			'avatargroups' => array(),
			'displayname' => $wtwconnect->escapeHTML($zrow["displayname"]),
			'avatardescription' => $wtwconnect->escapeHTML($zrow["avatardescription"]),
			'objectfolder' => $zrow["objectfolder"],
			'objectfile' => $zrow["objectfile"],
			'gender' => $zrow["gender"],
			'positionx' => $zrow["positionx"],
			'positiony' => $zrow["positiony"],
			'positionz' => $zrow["positionz"],
			'scalingx' => $zrow["scalingx"],
			'scalingy' => $zrow["scalingy"],
			'scalingz' => $zrow["scalingz"],
			'rotationx' => $zrow["rotationx"],
			'rotationy' => $zrow["rotationy"],
			'rotationz' => $zrow["rotationz"],
			'startframe' => $zrow["startframe"],
			'endframe' => $zrow["endframe"],
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"]),
			'snapshotid' => $zrow["snapshotid"],
			'shareuserid' => $zrow["shareuserid"],
			'alttag' => $wtwconnect->escapeHTML($zrow["alttag"]),
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'avatarparts'=> $zavatarparts,
			'avataranimationdefs'=> $zavataranimationdefs,
			'uploads'=> $zuploads,
			'files' => $zfiles,
			'contentratings'=>$zcontentratings,
			'users'=>$zusers
		);
	}
	
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-shareavatar.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: sound.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides sound file information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/sound.php");
	
	/* get values from querystring or session */
	$zsoundid = $wtwconnect->getVal('soundid','');

	/* select building molds that have been deleted */
	$zresults = $wtwconnect->query("
		select * from ".wtw_tableprefix."uploads
		where uploadid='".$zsoundid."'
		and deleted=0 limit 1;");

	header('Access-Control-Allow-Origin: *');
	foreach ($zresults as $zrow) {
		header('Content-Transfer-Encoding: binary');
		header('Accept-Ranges: bytes');
		header('Cache-Control: no-cache');
		header("Content-Length: ".$zrow["filesize"]);
		header("Content-type: ".$zrow["filetype"]);
		//header("Content-Disposition: attachment; filename=".$zrow["filename"]);
		echo $zrow["filedata"];
	}
} catch (Exception $e) {
	$wtwconnect->serror("connect-sound.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: thing.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Thing information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/thing.php");
	
	/* get values from querystring or session */
	$zthingid = $wtwconnect->getVal('thingid','');

	/* get thing */
	$zresults = $wtwconnect->query("
	select *,
		case when snapshotid = '' then ''
			else
				(select u1.filepath 
					from ".wtw_tableprefix."uploads u1 
					where u1.uploadid=snapshotid limit 1)
			end as snapshotpath
	from ".wtw_tableprefix."things
    where thingid='".$zthingid."'
       and deleted=0;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array('userid'=> $zrow["userid"]);
		$zthinginfo = array(
			'thingid' => $zrow["thingid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'thingname' => $wtwconnect->escapeHTML($zrow["thingname"]),
			'thingdescription' => $wtwconnect->escapeHTML($zrow["thingdescription"]),
			'analyticsid'=> $zrow["analyticsid"],
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["snapshotpath"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"]
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zresponse[$i] = array(
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'alttag'=> $zalttag,
			'authorizedusers'=> $zauthorizedusers
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-thing.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: thingmoldsrecover.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides thing mold information to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/thingmoldsrecover.php");
	
	/* get values from querystring or session */
	$zthingid = $wtwconnect->getVal('thingid','');
	$zthingind = $wtwconnect->getVal('thingind','-1');
	$zthingmoldid = $wtwconnect->getVal('thingmoldid','');
	$zconnectinggridid = $wtwconnect->getVal('connectinggridid','');
	$zconnectinggridind = $wtwconnect->getVal('connectinggridind','-1');

	/* select thing mold to recover */
	$zresults = $wtwconnect->query("
		select a1.*,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfolder 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfolder,
			case when a1.uploadobjectid = '' then ''
				else
					(select objectfile 
						from ".wtw_tableprefix."uploadobjects 
						where uploadobjects.uploadobjectid=a1.uploadobjectid limit 1)
				end as objectfile,
			case when a1.textureid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.textureid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.textureid limit 1)
					end 
				end as texturepath,
			case when a1.texturebumpid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
						(select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.originalid=u1.uploadid 
							where u2.uploadid=a1.texturebumpid limit 1)
						else (select u1.filepath 
							from ".wtw_tableprefix."uploads u2 
								left join ".wtw_tableprefix."uploads u1 
									on u2.websizeid=u1.uploadid 
							where u2.uploadid=a1.texturebumpid limit 1)
					end 
				end as texturebumppath,
			case when a1.heightmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.heightmapid limit 1)
					end 
				end as heightmappath,
			case when a1.mixmapid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.mixmapid limit 1)
					end 
				end as mixmappath,
			case when a1.texturerid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturerid limit 1)
					end 
				end as texturerpath,
			case when a1.texturegid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturegid limit 1)
					end 
				end as texturegpath,
			case when a1.texturebid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebid limit 1)
					end 
				end as texturebpath,
			case when a1.texturebumprid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumprid limit 1)
					end 
				end as texturebumprpath,
			case when a1.texturebumpgid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpgid limit 1)
					end 
				end as texturebumpgpath,
			case when a1.texturebumpbid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.texturebumpbid limit 1)
					end 
				end as texturebumpbpath,
			case when a1.videoid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.videoid limit 1)
				end as video,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select originalid 
								from ".wtw_tableprefix."uploads where uploadid=a1.videoposterid limit 1)
						else (select websizeid 
								from ".wtw_tableprefix."uploads where uploadid=a1.videoposterid limit 1)
					end 
				end as videoposterid,
			case when a1.videoposterid = '' then ''
				else
					case when a1.graphiclevel = '1' then 
							(select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.originalid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
						else (select u1.filepath 
								from ".wtw_tableprefix."uploads u2 
									left join ".wtw_tableprefix."uploads u1 
										on u2.websizeid=u1.uploadid 
								where u2.uploadid=a1.videoposterid limit 1)
					end 
				end as videoposter,
			case when a1.soundid = '' then ''
				else
					(select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=a1.soundid limit 1)
				end as soundpath,
			(select count(*) from ".wtw_tableprefix."thingmolds 
				where thingid='".$zthingid."' and csgmoldid=a1.thingmoldid) as csgcount
		from ".wtw_tableprefix."thingmolds a1
			left join ".wtw_tableprefix."things t1
				on a1.thingid = t1.thingid
		where a1.thingid='".$zthingid."'
		   and a1.thingmoldid='".$zthingmoldid."';");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	$zmolds = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zobjectanimations = null;
		$ztempwebtext = "";
		if ($wtwconnect->hasValue($zrow["webtext"])) {
			$ztempwebtext = implode('',(array)$zrow["webtext"]);
		}
		if ($wtwconnect->hasValue($zrow["uploadobjectid"])) {
			$zobjectanimations = $wtwconnect->getobjectanimations($zrow["uploadobjectid"]);
		}
		$zcommunityinfo = array(
			'communityid'=> '',
			'communityind'=> '',
			'analyticsid'=> ''
		);
		$zbuildinginfo = array(
			'buildingid'=> '',
			'buildingind'=> '',
			'analyticsid'=> ''
		);
		$zthinginfo = array(
			'thingid'=> $zrow["thingid"],
			'thingind'=> $zthingind,
			'analyticsid'=> $zrow["analyticsid"]
		);
		$zposition = array(
			'x'=> $zrow["positionx"], 
			'y'=> $zrow["positiony"], 
			'z'=> $zrow["positionz"],
			'scroll'=>''
		);
		$zscaling = array(
			'x'=> $zrow["scalingx"], 
			'y'=> $zrow["scalingy"], 
			'z'=> $zrow["scalingz"],
			'special1'=> $zrow["special1"],
			'special2'=> $zrow["special2"]
		);
		$zrotation = array(
			'x'=> $zrow["rotationx"], 
			'y'=> $zrow["rotationy"], 
			'z'=> $zrow["rotationz"],
			'billboard'=> $zrow["billboard"]
		);
		$zcsg = array(
			'moldid'=> $zrow["csgmoldid"], 
			'moldind'=>'-1', 
			'action'=> $zrow["csgaction"], 
			'count'=> $zrow["csgcount"] 
		);
		$zobjects = array(
			'uploadobjectid'=> $zrow["uploadobjectid"], 
			'folder'=> $zrow["objectfolder"], 
			'file'=> $zrow["objectfile"],
			'objectanimations'=> $zobjectanimations,
			'light'=> '',
			'shadows'=> ''
		);
		$zgraphics = array(
			'texture'=> array(
				'id'=> $zrow["textureid"],
				'path'=> $zrow["texturepath"],
				'bumpid'=> $zrow["texturebumpid"],
				'bumppath'=> $zrow["texturebumppath"],
				'videoid'=> $zrow["videoid"],
				'video'=> $zrow["video"],
				'videoposterid'=> $zrow["videoposterid"],
				'videoposter'=> $zrow["videoposter"],
				'backupid'=> ''
			),
			'heightmap'=> array(
				'original'=> '',
				'id'=> $zrow["heightmapid"],
				'path'=> $zrow["heightmappath"],
				'minheight'=> $zrow["minheight"],
				'maxheight'=> $zrow["maxheight"],
				'mixmapid'=> $zrow["mixmapid"],
				'mixmappath'=> $zrow["mixmappath"],
				'texturerid'=> $zrow["texturerid"],
				'texturerpath'=> $zrow["texturerpath"],
				'texturegid'=> $zrow["texturegid"],
				'texturegpath'=> $zrow["texturegpath"],
				'texturebid'=> $zrow["texturebid"],
				'texturebpath'=> $zrow["texturebpath"],
				'texturebumprid'=> $zrow["texturebumprid"],
				'texturebumprpath'=> $zrow["texturebumprpath"],
				'texturebumpgid'=> $zrow["texturebumpgid"],
				'texturebumpgpath'=> $zrow["texturebumpgpath"],
				'texturebumpbid'=> $zrow["texturebumpbid"],
				'texturebumpbpath'=> $zrow["texturebumpbpath"]
			),
			'uoffset'=> $zrow["uoffset"],
			'voffset'=> $zrow["voffset"],
			'uscale'=> $zrow["uscale"],
			'vscale'=> $zrow["vscale"],
			'level'=> $zrow["graphiclevel"],
			'receiveshadows'=> $zrow["receiveshadows"],
			'castshadows'=> $zrow["castshadows"],
			'waterreflection'=> $zrow["waterreflection"], 
			'webimages'=> $wtwconnect->getwebimages($zrow["thingmoldid"], "", "",-1)
		);
		$zwebtext = array(
			'webtext'=> $zrow["webtext"],
			'fullheight'=> '0',
			'scrollpos'=> '0',
			'webstyle'=> $zrow["webstyle"]
		);
		$zalttag = array(
			'name' => $zrow["alttag"]
		);
		$zpaths = array(
			'path1'=> $wtwconnect->getmoldpoints($zrow["thingmoldid"], '', '', 1, $zrow["shape"]),
			'path2'=> $wtwconnect->getmoldpoints($zrow["thingmoldid"], '', '', 2, $zrow["shape"])
		);
		$zcolor = array(
			'diffusecolor'=> $zrow["diffusecolor"],
			'emissivecolor'=> $zrow["emissivecolor"],
			'specularcolor'=> $zrow["specularcolor"],
			'ambientcolor'=> $zrow["ambientcolor"]
		);
		$zsound = array(
			'id' => $zrow["soundid"],
			'path' => $zrow["soundpath"],
			'name' => $zrow["soundname"],
			'attenuation' => $zrow["soundattenuation"],
			'loop' => $zrow["soundloop"],
			'maxdistance' => $zrow["soundmaxdistance"],
			'rollofffactor' => $zrow["soundrollofffactor"],
			'refdistance' => $zrow["soundrefdistance"],
			'coneinnerangle' => $zrow["soundconeinnerangle"],
			'coneouterangle' => $zrow["soundconeouterangle"],
			'coneoutergain' => $zrow["soundconeoutergain"],
			'sound' => ''
		);
		$zphysics = array(
			'enabled'=>$zrow["physicsenabled"],
			'center'=>array(
				'x'=>$zrow["physicscenterx"],
				'y'=>$zrow["physicscentery"],
				'z'=>$zrow["physicscenterz"]
			),
			'extents'=>array(
				'x'=>$zrow["physicsextentsx"],
				'y'=>$zrow["physicsextentsy"],
				'z'=>$zrow["physicsextentsz"]
			),
			'friction'=>$zrow["physicsfriction"],
			'istriggershape'=>$zrow["physicsistriggershape"],
			'mass'=>$zrow["physicsmass"],
			'pointa'=>array(
				'x'=>$zrow["physicspointax"],
				'y'=>$zrow["physicspointay"],
				'z'=>$zrow["physicspointaz"]
			),
			'pointb'=>array(
				'x'=>$zrow["physicspointbx"],
				'y'=>$zrow["physicspointby"],
				'z'=>$zrow["physicspointbz"]
			),
			'radius'=>$zrow["physicsradius"],
			'restitution'=>$zrow["physicsrestitution"],
			'rotation'=>array(
				'x'=>$zrow["physicsrotationx"],
				'y'=>$zrow["physicsrotationy"],
				'z'=>$zrow["physicsrotationz"],
				'w'=>$zrow["physicsrotationw"]
			),
			'startasleep'=>$zrow["physicsstartasleep"]
		);
		$zmolds[$i] = array(
			'communityinfo'=> $zcommunityinfo, 
			'buildinginfo'=> $zbuildinginfo, 
			'thinginfo'=> $zthinginfo,  
			'moldid'=> $zrow["thingmoldid"], 
			'moldind'=> '-1',
			'shape'=> $zrow["shape"], 
			'covering'=> $zrow["covering"], 
			'position'=> $zposition,
			'scaling'=> $zscaling,
			'rotation'=> $zrotation,
			'csg'=> $zcsg,
			'objects'=> $zobjects,
			'graphics'=> $zgraphics, 
			'webtext'=> $zwebtext, 
			'alttag'=> $zalttag,
			'paths'=> $zpaths,
			'color'=> $zcolor,
			'sound'=> $zsound,
			'physics'=> $zphysics,
			'subdivisions'=> $zrow["subdivisions"], 
			'subdivisionsshown'=>'0',
			'shown'=>'0',
			'opacity'=> $zrow["opacity"], 
			'checkcollisions'=> $zrow["checkcollisions"], 
			'ispickable'=> $zrow["ispickable"], 
			'jsfunction'=> '',
			'jsparameters'=> '',
			'actionzoneid'=> $zrow["actionzoneid"],
			'actionzone2id'=> $zrow["actionzone2id"],
			'loadactionzoneid'=> $zrow["loadactionzoneid"],
			'unloadactionzoneid'=> $zrow["unloadactionzoneid"],
			'connectinggridid'=> $zconnectinggridid,
			'connectinggridind'=> $zconnectinggridind,
			'attachmoldind'=> '-1',
			'loaded'=> '0',
			'parentname'=>'',
			'moldname'=>'');
		$i += 1;
	}
	$zresponse['molds'] = $zmolds;
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-thingmoldsrecover.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: thingnames.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides a list of 3D Thing Names information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/thingnames.php");
	
	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;
	
	/* select building molds that have been deleted */
	$zresults = $wtwconnect->query("
		select t1.*
		from ".wtw_tableprefix."things t1 
		 where t1.deleted=0
		order by t1.thingname,t1.thingid;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zthings = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zthings[$i] = array(
			'thingid' => $zrow["thingid"],
			'thingname' => $wtwconnect->escapeHTML($zrow["thingname"]),
			'thingdescription' => $wtwconnect->escapeHTML($zrow["thingdescription"])
		);
		$i += 1;
	}
	echo json_encode($zthings);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-thingnames.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: thingrecoveritems.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides thing mold information list to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/thingrecoveritems.php");
	
	/* get values from querystring or session */
	$zthingid = $wtwconnect->getVal('thingid','');

	/* get thing molds that have been deleted */
	$zresults = $wtwconnect->query("
		select shape as item,
			thingmoldid as itemid,
			'thingmolds' as itemtype
		from ".wtw_tableprefix."thingmolds
		where thingid='".$zthingid."'
		   and deleted>0
		   and not deleteddate is null
		order by deleteddate desc, 
			thingmoldid desc;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zresponse[$i] = array(
			'itemid'=> $zrow["itemid"], 
			'item'=> $zrow["item"],
			'itemtype'=> $zrow["itemtype"],
			'parentname'=>'');
		$i += 1;
	}
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-thingrecoveritems.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: things.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides 3D Thing information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/things.php");
	
	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;
	$zfilter = $wtwconnect->getVal('filter','mine');
	
	/* check user for global roles with access */
	$hasaccess = false;
	if ($zfilter == 'all') {
		$zroles = $wtwconnect->getUserRoles($zuserid);
		foreach ($zroles as $zrole) {
			if (strtolower($zrole['rolename']) == 'admin' || strtolower($zrole['rolename']) == 'architect' || strtolower($zrole['rolename']) == 'developer' || strtolower($zrole['rolename']) == 'graphics artist') {
				$hasaccess = true;
			}
		}
	}
	/* select things by userid */
	$zresults = array();
	if ($hasaccess) {
		/* select things based on global access */
		$zresults = $wtwconnect->query("
			select distinct t1.*,
				u1.filetype,
				u1.filepath,
				u1.filedata
			from ".wtw_tableprefix."things t1 
				left join ".wtw_tableprefix."uploads u1
					on t1.snapshotid=u1.uploadid
			 where t1.deleted=0
			order by t1.thingname,t1.thingid;");
	} else {
		/* select things based on granular user permissions */
		$zresults = $wtwconnect->query("
			select distinct t1.*,
				u1.filetype,
				u1.filepath,
				u1.filedata
			from ".wtw_tableprefix."things t1 
				inner join ".wtw_tableprefix."userauthorizations ua1
					on t1.thingid=ua1.thingid
				left join ".wtw_tableprefix."uploads u1
					on t1.snapshotid=u1.uploadid
			 where ua1.userid='".$zuserid."'
				and not ua1.thingid=''
				and t1.deleted=0
				and ua1.deleted=0
				and (ua1.useraccess='admin'
					or ua1.useraccess='architect')
			order by t1.thingname,t1.thingid;");
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zauthorizedusers = array('userid'=> $zrow["userid"]);
		$snapshotdata = null;
		if ((!isset($zrow["filepath"]) || empty($zrow["filepath"])) && isset($zrow["filedata"]) && !empty($zrow["filedata"])) {
			$snapshotdata = "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["filedata"]));
		}
		$zthinginfo = array(
			'thingid' => $zrow["thingid"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $wtwconnect->escapeHTML($zrow["versiondesc"]),
			'thingname' => $wtwconnect->escapeHTML($zrow["thingname"]),
			'thingdescription' => $wtwconnect->escapeHTML($zrow["thingdescription"]),
			'snapshotid' => $zrow["snapshotid"],
			'snapshotpath' => $zrow["filepath"],
			'analyticsid'=> $zrow["analyticsid"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'snapshotdata'=> $snapshotdata
		);
		$zshare = array(
			'templatename' => $wtwconnect->escapeHTML($zrow["templatename"]),
			'description' => $wtwconnect->escapeHTML($zrow["description"]),
			'tags' => $wtwconnect->escapeHTML($zrow["tags"])
		);
		$zalttag = array(
			'name' => $wtwconnect->escapeHTML($zrow["alttag"])
		);
		$zresponse[$i] = array(
			'thinginfo'=> $zthinginfo,
			'serverfranchiseid' => '',
			'share'=> $zshare,
			'alttag'=> $zalttag,
			'authorizedusers'=> $zauthorizedusers
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-things.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: upload.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides upload file information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/upload.php");
	
	/* get values from querystring or session */
	$zuploadid = $wtwconnect->getVal('uploadid','');
	$setid = $wtwconnect->getVal('setid','');

	/* select upload file data */
	$zresults = $wtwconnect->query("
		select * 
		from ".wtw_tableprefix."uploads
		where uploadid='".$zuploadid."'
		and deleted=0;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zuploadinfo = array(
			'title' => $zrow["filetitle"],
			'name' => $zrow["filename"],
			'extension' => $zrow["fileextension"],
			'type' => $zrow["filetype"],
			'size' => $zrow["filesize"],
			'width' => $zrow["imagewidth"],
			'height' => $zrow["imageheight"]
		);
		$zresponse[$i] = array(
			'uploadinfo'=> $zuploadinfo,
			'id'=> $zrow["uploadid"],
			'uploadid'=> $zrow["uploadid"],
			'originalid'=> $zrow["originalid"],
			'websizeid'=> $zrow["websizeid"],
			'thumbnailid'=> $zrow["thumbnailid"],
			'filepath'=> $zrow["filepath"],
			'data'=> "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["filedata"])),
			'dataaudio'=> addslashes(base64_encode($zrow["filedata"])),
			'userid'=> $zrow["userid"],
			'queue'=> '1',
			'setid'=> $setid
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-upload.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: uploadmedia.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides uploaded media information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/uploadmedia.php");
	
	/* get values from querystring or session */
	$zuploadid = $wtwconnect->getVal('uploadid','');

	/* select upload file data, return more info */
	$zresults = $wtwconnect->query("
		select u1.*,
			u2.uploadid as originalid2,
			u2.filepath as originalpath,
			u2.filedata as originaldata,
			u2.filesize as originalsize,
			u2.imagewidth as originalwidth,
			u2.imageheight as originalheight,
			u3.uploadid as websizeid2,
			u3.filepath as websizepath,
			u3.filedata as websizedata,
			u3.filesize as websizesize,
			u3.imagewidth as websizewidth,
			u3.imageheight as websizeheight
		from ".wtw_tableprefix."uploads u1 
			left join ".wtw_tableprefix."uploads u2
				on u1.originalid=u2.uploadid
			left join ".wtw_tableprefix."uploads u3
				on u1.websizeid=u3.uploadid
		where u1.uploadid='".$zuploadid."'
		and u1.deleted=0;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$websize = array(
			'id'=> '',
			'path' => '',
			'size' => '',
			'width' => '',
			'height' => '',
			'data'=> ''
		);
		$original = array(
			'id'=> '',
			'path' => '',
			'size' => '',
			'width' => '',
			'height' => '',
			'data'=> ''
		);
		$zuploadinfo = array(
			'title' => $zrow["filetitle"],
			'name' => $zrow["filename"],
			'extension' => $zrow["fileextension"],
			'type' => $zrow["filetype"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'deleteddate' => $zrow["deleteddate"],
			'deleteduserid' => $zrow["deleteduserid"],
			'hide' => $zrow["hide"],
			'hideuserid' => $zrow["hideuserid"],
			'hidedate' => $zrow["hidedate"],
			'stock' => $zrow["stock"]
		);
		$thumbnail = array(
			'id'=> $zrow["thumbnailid"],
			'path' => $zrow["filepath"],
			'size' => $zrow["filesize"],
			'width' => $zrow["imagewidth"],
			'height' => $zrow["imageheight"],
			'data'=> "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["filedata"])),
			'dataaudio'=> addslashes(base64_encode($zrow["filedata"]))
		);
		if (!empty($zrow["websizeid2"])) {
			$websize = array(
				'id'=> $zrow["websizeid"],
				'path' => $zrow["websizepath"],
				'size' => $zrow["websizesize"],
				'width' => $zrow["websizewidth"],
				'height' => $zrow["websizeheight"],
				'data'=> "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["websizedata"]))
			);
		}
		if (!empty($zrow["originalid2"])) {
			$original = array(
				'id'=> $zrow["originalid"],
				'path' => $zrow["originalpath"],
				'size' => $zrow["originalsize"],
				'width' => $zrow["originalwidth"],
				'height' => $zrow["originalheight"],
				'data'=> "data:".$zrow["filetype"].";base64,".addslashes(base64_encode($zrow["originaldata"]))
			);
		}
		$zresponse[$i] = array(
			'uploadinfo'=> $zuploadinfo,
			'id'=> $zrow["uploadid"],
			'uploadid'=> $zrow["uploadid"],
			'thumbnail'=> $thumbnail,
			'websize'=> $websize,
			'original'=> $original,
			'userid'=> $zrow["userid"]
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-uploadmedia.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: user.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic user information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/user.php");
	
	$zuserid = $wtwconnect->getVal('userid','');

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$zuser = array();
	
	if ($wtwconnect->isUserInRole("admin")) {
		/* get users information */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."users
			where deleted=0
				and userid='".$zuserid."'
			limit 1;");
		
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			$usertokenexists = 'false';
			if ($wtwconnect->hasValue($zrow["usertoken"])) {
				$usertokenexists = 'true';
			}
			$zuser = array(
				'userid' => $zrow["userid"],
				'uploadpathid' => $zrow["uploadpathid"],
				'userimageurl' => $zrow["userimageurl"],
				'email' => $zrow["email"],
				'displayname' => $zrow["displayname"],
				'firstname' => $zrow["firstname"],
				'lastname' => $zrow["lastname"],
				'gender' => $zrow["gender"],
				'dob' => $zrow["dob"],
				'usertoken' => $usertokenexists,
				'createdate' => $zrow["createdate"],
				'createuserid' => $zrow["createuserid"],
				'updatedate' => $zrow["updatedate"],
				'updateuserid' => $zrow["updateuserid"],
				'deleteddate' => $zrow["deleteddate"],
				'deleteduserid' => $zrow["deleteduserid"],
				'deleted' => $zrow["deleted"],
				'roles' => $wtwconnect->getUserRoles($zrow["userid"])
				);
		}
	}
	echo json_encode($zuser);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-user.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: useraccess.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides user access information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/useraccess.php");
	
	/* get values from querystring or session */
	$zcommunityid = $wtwconnect->getVal('communityid','');
	$zbuildingid = $wtwconnect->getVal('buildingid','');
	$zthingid = $wtwconnect->getVal('thingid','');
	$zuserid = $wtwconnect->userid;
	$hasaccess = "";
    
	/* get is user has access to community, building, or thing */
	$zresults = $wtwconnect->query("
		select '1' as hasaccess 
		from ".wtw_tableprefix."userauthorizations 
		where userid='".$zuserid."' 
			and communityid='".$zcommunityid."' 
			and buildingid='".$zbuildingid."' 
			and thingid='".$zthingid."' 
			and deleted=0 
			and not useraccess='browse'
        limit 1");
	foreach ($zresults as $zrow) {
		$hasaccess = $zrow["hasaccess"];
	}
	$zresults = array();
	if ($wtwconnect->hasValue($hasaccess)) {
		$zresults = $wtwconnect->query("
			select ua1.userauthorizationid,
				u1.email,
				u1.displayname,
				ua1.userid,
				ua1.useraccess
			from ".wtw_tableprefix."userauthorizations ua1
				left join ".wtw_tableprefix."users u1
					on ua1.userid=u1.userid
			where ua1.buildingid='".$zbuildingid."'
				and ua1.communityid='".$zcommunityid."'
				and ua1.thingid='".$zthingid."'
				and ua1.deleted=0
				and (u1.deleted=0 or u1.deleted is null)
			order by u1.displayname,ua1.userid,ua1.userauthorizationid;");
	} else {
		$zresults = $wtwconnect->query("
			select userauthorizationid,
				'' as email,
				'' as displayname,
				userid,
				useraccess
			from ".wtw_tableprefix."userauthorizations
			where buildingid='".$zbuildingid."'
				and communityid='".$zcommunityid."'
				and thingid='".$zthingid."'
				and deleted=0
			order by userid,userauthorizationid;");
	}

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$suser = array(
			'userauthorizationid' => $zrow["userauthorizationid"],
			'email' => $zrow["email"],
			'displayname' => $zrow["displayname"],
			'userid' => $zrow["userid"],
			'useraccess' => $zrow["useraccess"]);
		$zresponse[$i] = $suser;
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-useraccess.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: userauthenticate.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic user information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/userauthenticate.php");

	$zuseremail = $wtwconnect->decode64($wtwconnect->getPost('useremail',''));
	$zuserpassword = $wtwconnect->decode64($wtwconnect->getPost('password',''));
	$zwpinstanceid = $wtwconnect->decode64($wtwconnect->getPost('wpinstanceid',''));
	$zwebsiteurl = $wtwconnect->decode64($wtwconnect->getPost('websiteurl',''));
	$zserverip = $wtwconnect->decode64($wtwconnect->getPost('serverip',''));
	$zwordpresstoken = '';
	$zfound = false;

	echo $wtwconnect->addConnectHeader('*');
	
	$zresponse = array(
		'userid' => '',
		'wordpresstoken' => '',
		'serror' => 'Invalid Login.'
	);
	
	/* get users information */
	$zresults = $wtwconnect->query("
		select u1.* 
		from ".wtw_tableprefix."users u1
			inner join ".wtw_tableprefix."usersinroles uir1
			on u1.userid=uir1.userid
			inner join ".wtw_tableprefix."roles r1
			on uir1.roleid=r1.roleid
		where r1.rolename='Admin'
			and r1.deleted=0
			and u1.deleted=0
			and u1.email='".$zuseremail."';");
	
	/* check for account and verify password */
	foreach ($zresults as $zrow) {
		$zuserid = $zrow["userid"];
		$zpasswordhash = $zrow["userpassword"];
		if ($zfound == false && password_verify($zuserpassword, $zpasswordhash)) {
			$zfound = true;
			/* user is local on server and is valid login */
			$zwordpresstoken = $zrow["wordpresstoken"];
			if (!isset($zwordpresstoken) || empty($zwordpresstoken)) {
				$zwordpresstoken = base64_encode($wtwconnect->getRandomString(128,1));
				$wtwconnect->query("
					update ".wtw_tableprefix."users
					set wordpresstoken='".$zwordpresstoken."'
					where deleted=0
						and userid='".$zuserid."';");
			}
			/* format json return dataset */
			$zresponse = array(
				'userid' => $zuserid,
				'wordpresstoken' => $zwordpresstoken,
				'serror' => ''
			);
		} else if ($zfound == false) {
			/* user is global and requires global validation */
			$zpostdata = stream_context_create(array('http' =>
				array(
					'method'  => 'POST',
					'header'  => 'Content-Type: application/x-www-form-urlencoded',
					'content' => http_build_query(
						array(
							'useremail' => base64_encode($zuseremail),
							'password' => base64_encode($zuserpassword),
							'wpinstanceid' => base64_encode($zwpinstanceid),
							'websiteurl' => base64_encode($zwebsiteurl),
							'serverip' => base64_encode($zserverip),
							'function' => 'login'
						)
					)
				)
			));
				
			$zresults = $wtwconnect->openFilefromURL('https://3dnet.walktheweb.com/connect/authenticate.php', false, $zpostdata);			
			if ($wtwconnect->hasValue($zresults)) {
				$zresults = json_decode($zresults);
			}

			if ($wtwconnect->hasValue($zresults->usertoken)) {
				/* authenticated */
				$zfound = true;
				$zwordpresstoken = $zrow["wordpresstoken"];
				if (!isset($zwordpresstoken) || empty($zwordpresstoken)) {
					$zwordpresstoken = base64_encode($wtwconnect->getRandomString(128,1));
					$wtwconnect->query("
						update ".wtw_tableprefix."users
						set wordpresstoken='".$zwordpresstoken."',
							usertoken='".base64_encode($zresults->usertoken)."',
							updatedate=now(),
							updateuserid='".$zuserid."'
						where deleted=0
							and userid='".$zuserid."';");
				} else {
					$wtwconnect->query("
						update ".wtw_tableprefix."users
						set usertoken='".base64_encode($zresults->usertoken)."',
							updatedate=now(),
							updateuserid='".$zuserid."'
						where deleted=0
							and userid='".$zuserid."';");
				}
				$zresponse = array(
					'userid' => $zuserid,
					'wordpresstoken' => $zwordpresstoken,
					'serror' => ''
				);
			} else {
				/* did not authenticate, return error */
				$zresponse = $zresults;
			}
		}
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-userauthenticate.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: useravatar.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides local user's avatar information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/useravatar.php");

	/* get values from querystring or session */
	$zinstanceid = $wtwconnect->decode64($wtwconnect->getVal('instanceid',''));
	$zuserid = $wtwconnect->decode64($wtwconnect->getVal('userid',''));
	$zuserip = $wtwconnect->decode64($wtwconnect->getVal('userip',''));
	$zuseravatarid = $wtwconnect->decode64($wtwconnect->getVal('useravatarid',''));
	$zavatarid = $wtwconnect->decode64($wtwconnect->getVal('avatarid',''));
	$zglobalhash = $wtwconnect->getVal('globalhash','');
	
	$zfounduseravatarid = '';
	$zfoundavatarid = '';

	if ($wtwconnect->hasValue($zuseravatarid)) {
		/* check for user avatar */
		$zresults = $wtwconnect->query("
			select useravatarid, avatarid 
			from ".wtw_tableprefix."useravatars 
			where useravatarid='".$zuseravatarid."' 
				and deleted=0
			limit 1;");
		foreach ($zresults as $zrow) {
			$zfounduseravatarid = $zrow["useravatarid"];
			$zfoundavatarid = $zrow["avatarid"];
		}
	}

	if ((!isset($zfounduseravatarid) || empty($zfounduseravatarid)) && (!isset($zfoundavatarid) || empty($zfoundavatarid)) && isset($zuserid) && !empty($zuserid)) {
		/* check for user avatar for logged in user (latest used) */
		$zresults = $wtwconnect->query("
			select useravatarid, avatarid
			from ".wtw_tableprefix."useravatars 
			where userid='".$zuserid."' 
				and deleted=0 
			order by updatedate desc 
			limit 1;");
		foreach ($zresults as $zrow) {
			$zfounduseravatarid = $zrow["useravatarid"];
			$zfoundavatarid = $zrow["avatarid"];
		}
	}

	if ((!isset($zfounduseravatarid) || empty($zfounduseravatarid)) && (!isset($zfoundavatarid) || empty($zfoundavatarid)) && isset($zavatarid) && !empty($zavatarid)) {
		/* check for avatar selected */
		$zresults = $wtwconnect->query("
			select avatarid
			from ".wtw_tableprefix."avatars 
			where avatarid='".$zavatarid."' 
				and deleted=0 
			limit 1;");
		foreach ($zresults as $zrow) {
			$zfoundavatarid = $zrow["avatarid"];
		}
	}

	if ((!isset($zfounduseravatarid) || empty($zfounduseravatarid)) && (!isset($zfoundavatarid) || empty($zfoundavatarid)) && isset($zinstanceid) && !empty($zinstanceid)) {
		/* check for user avatar for by instance (latest used) */
		$zresults = $wtwconnect->query("
			select useravatarid, avatarid
			from ".wtw_tableprefix."useravatars 
			where instanceid='".$zinstanceid."' 
				and deleted=0 
			order by updatedate desc 
			limit 1;");
		foreach ($zresults as $zrow) {
			$zfounduseravatarid = $zrow["useravatarid"];
			$zfoundavatarid = $zrow["avatarid"];
		}
	}

	$i = 0;
	$zavatar = array();
	$zavatarparts = array();
	$zfiles = array();
	$zusers = array();
	$zresponse = null;
	$zavataranimationdefs = array();
	$zcontentratings = array();
	$zblockedby = array();
	$zbannedby = array();
	$zpositionx = '0.00';
	$zpositiony = '0.00';
	$zpositionz = '0.00';
	$zscalingx = '1.0000';
	$zscalingy = '1.0000';
	$zscalingz = '1.0000';
	$zrotationx = '0.00';
	$zrotationy = '0.00';
	$zrotationz = '0.00';
	$zobjectfolder = "";
	$zobjectfile = "";
	$zstartframe = '1';
	$zendframe = '1';
	$zgender = 'female';
	$zdisplayname = 'Anonymous';
	$zavatardescription = '';
	$zavatargroup = 'Custom';
	$zanonymous = '0';
	$zprivacy = 0;
	$zversionid = '';
	$zversion = '1.0.0';
	$zversionorder = 1000000;
	$zversiondesc = 'Initial Version';
	
	$zwalkspeed = '1';
	$zwalkanimationspeed = '1';
	$zturnspeed = '1';
	$zturnanimationspeed = '1';
	
	$zenteranimation = "1";
	$zexitanimation = "1";
	$zenteranimationparameter = "";
	$zexitanimationparameter = "";

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	if ($wtwconnect->hasValue($zfounduseravatarid)) {
		/* get user avatar */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."useravatars 
			where useravatarid='".$zfounduseravatarid."' 
				and deleted=0
			limit 1;");
		foreach ($zresults as $zrow) {
			if ($zrow["userid"] == '') {
				$zanonymous = '1';
			}
			$zuseravatarid = $zrow["useravatarid"];
			$zinstanceid = $zrow["instanceid"];
			$zavatarid = $zrow["avatarid"];
			$zfoundavatarid = $zrow["avatarid"];
			$zuserid = $zrow["userid"];
			$zversionid = $zrow["versionid"];
			$zversion = $zrow["version"];
			$zversionorder = $zrow["versionorder"];
			$zversiondesc = $zrow["versiondesc"];
			$zobjectfolder = $zrow["objectfolder"];
			$zobjectfile = $zrow["objectfile"];
			$zstartframe = $zrow["startframe"];
			$zendframe = $zrow["endframe"];
			$zwalkspeed = $zrow["walkspeed"];
			$zwalkanimationspeed = $zrow["walkanimationspeed"];
			$zturnspeed = $zrow["turnspeed"];
			$zturnanimationspeed = $zrow["turnanimationspeed"];
			$zdisplayname = $zrow["displayname"];
			$zavatardescription = $zrow["avatardescription"];
			$zgender = $zrow["gender"];
			$zavatargroup = $zrow["avatargroup"];
			$zprivacy = $zrow["privacy"];
			$zenteranimation = $zrow["enteranimation"];
			$zexitanimation = $zrow["exitanimation"];
			$zenteranimationparameter = $zrow["enteranimationparameter"];
			$zexitanimationparameter = $zrow["exitanimationparameter"];
			$zpositionx = $zrow["positionx"];
			$zpositiony = $zrow["positiony"];
			$zpositionz = $zrow["positionz"];
			$zscalingx = $zrow["scalingx"];
			$zscalingy = $zrow["scalingy"];
			$zscalingz = $zrow["scalingz"];
			$zrotationx = $zrow["rotationx"];
			$zrotationy = $zrow["rotationy"];
			$zrotationz = $zrow["rotationz"];
			$zobjectfolder = $zrow["objectfolder"];
			$zobjectfile = $zrow["objectfile"];
			$zstartframe = $zrow["startframe"];
			$zendframe = $zrow["endframe"];
			$zwalkspeed = $zrow["walkspeed"];
			$zwalkanimationspeed = $zrow["walkanimationspeed"];
			$zturnspeed = $zrow["turnspeed"];
			$zturnanimationspeed = $zrow["turnanimationspeed"];
		}

		/* get avatar and color settings */
		$i = 0;
		$zresults = $wtwconnect->query("
			select avatarpartid,
				avatarpart,
				diffusecolor,
				specularcolor,
				emissivecolor,
				ambientcolor
			from ".wtw_tableprefix."useravatarcolors 
			where useravatarid='".$zfounduseravatarid."'
				and deleted=0
			order by avatarpart, updatedate desc, avatarpartid;");
		foreach ($zresults as $zrow) {
			$zavatarparts[$i] = array(
				'globalpartid'=>'',
				'avatarpartid'=> $zrow["avatarpartid"],
				'avatarpart'=> $zrow["avatarpart"],
				'diffusecolor'=> $zrow["diffusecolor"],
				'emissivecolor'=> $zrow["emissivecolor"],
				'specularcolor'=> $zrow["specularcolor"],
				'ambientcolor'=> $zrow["ambientcolor"]
			);
			$i += 1;
		}
		
		/* get the user avatar animations (the wait idle is stored in the user avatars table) */
		$zavataranimationdefs[0] = array(
			'animationind'=> 0,
			'useravataranimationid'=> '',
			'avataranimationid'=> '',
			'animationevent'=> 'onwait',
			'animationfriendlyname'=> 'Wait',
			'loadpriority'=> 100,
			'animationicon'=> '',
			'defaultspeedratio'=> 1.00,
			'speedratio'=> 1.00,
			'objectfolder'=> $zobjectfolder,
			'objectfile'=> $zobjectfile,
			'startframe'=> $zstartframe,
			'endframe'=> $zendframe,
			'animationloop'=> 1,
			'soundid'=> '',
			'soundmaxdistance'=> 100,
			'walkspeed'=> '1',
			'totalframes' => '0',
			'totalstartframe' => '0',
			'totalendframe' => '0'
		);
		/* get the user avatar animations (the rest are stored in the useravataranimations table) */
		$i = 1;
		$zresults = $wtwconnect->query("
			select a1.* 
				from ".wtw_tableprefix."useravataranimations a1
				inner join (
					select animationevent, max(updatedate) as updatedate, max(useravataranimationid) as useravataranimationid 
					from ".wtw_tableprefix."useravataranimations 
					where useravatarid='".$zfounduseravatarid."'
						and deleted=0
						and not animationevent='onoption'
					group by animationevent) a2
				on a1.useravataranimationid = a2.useravataranimationid
				where a1.useravatarid='".$zfounduseravatarid."' 
					and a1.deleted=0
			union
			select a3.* 
				from ".wtw_tableprefix."useravataranimations a3
				inner join (
					select animationfriendlyname, max(updatedate) as updatedate, max(useravataranimationid) as useravataranimationid 
					from ".wtw_tableprefix."useravataranimations 
					where useravatarid='".$zfounduseravatarid."'
						and deleted=0
						and animationevent='onoption'
					group by animationfriendlyname) a4
				on a3.useravataranimationid = a4.useravataranimationid
				where a3.useravatarid='".$zfounduseravatarid."' 
					and a3.deleted=0
			order by loadpriority desc, animationevent, animationfriendlyname, useravataranimationid;");
		foreach ($zresults as $zrow) {
			if (!empty($zrow["endframe"])) {
				$zavataranimationdefs[$i] = array(
					'animationind'=> $i,
					'useravataranimationid'=> $zrow["useravataranimationid"],
					'avataranimationid'=> $zrow["avataranimationid"],
					'animationevent'=> $zrow["animationevent"],
					'animationfriendlyname'=> addslashes($zrow["animationfriendlyname"]),
					'loadpriority'=> $zrow["loadpriority"],
					'animationicon'=> $zrow["animationicon"],
					'defaultspeedratio'=> $zrow["speedratio"],
					'speedratio'=> $zrow["speedratio"],
					'objectfolder'=> $zrow["objectfolder"],
					'objectfile'=> $zrow["objectfile"],
					'startframe'=> $zrow["startframe"],
					'endframe'=> $zrow["endframe"],
					'animationloop'=> $zrow["animationloop"],
					'walkspeed'=> $zrow["walkspeed"],
					'soundid'=> $zrow["soundid"],
					'soundmaxdistance'=> $zrow["soundmaxdistance"],
					'totalframes' => '0',
					'totalstartframe' => '0',
					'totalendframe' => '0'
				);
				$i += 1;
			}
		}
	} else {
		if (!isset($zfoundavatarid) || empty($zfoundavatarid)) {
			/* get the first anonymous avatar available (latest updated) */
			$zresults = $wtwconnect->query("
				select avatarid
				from ".wtw_tableprefix."avatars 
				where avatargroup='Anonymous' 
					and deleted=0 
				order by updatedate desc
				limit 1;");
			foreach ($zresults as $zrow) {
				$zfoundavatarid = $zrow["avatarid"];
			}
		}
		
		/* get avatar by avatarid */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."avatars 
			where avatarid='".$zfoundavatarid."'
				and deleted=0 
			limit 1;");
		foreach ($zresults as $zrow) {
			$zuseravatarid = '';
			$zanonymous = '1';
			$zversionid = $zrow["versionid"];
			$zversion = $zrow["version"];
			$zversionorder = $zrow["versionorder"];
			$zversiondesc = $zrow["versiondesc"];
			$zpositionx = $zrow["positionx"];
			$zpositiony = $zrow["positiony"];
			$zpositionz = $zrow["positionz"];
			$zscalingx = $zrow["scalingx"];
			$zscalingy = $zrow["scalingy"];
			$zscalingz = $zrow["scalingz"];
			$zrotationx = $zrow["rotationx"];
			$zrotationy = $zrow["rotationy"];
			$zrotationz = $zrow["rotationz"];
			$zobjectfolder = $zrow["objectfolder"];
			$zobjectfile = $zrow["objectfile"];
			$zstartframe = $zrow["startframe"];
			$zendframe = $zrow["endframe"];
			$zdisplayname = 'Anonymous';
			$zavatardescription = $zrow["avatardescription"];
			$zavatargroup = $zrow["avatargroup"];
			$zgender = $zrow["gender"];
			$zprivacy = '0';
			$zenteranimation = '0';
			$zexitanimation = '0';
			$zenteranimationparameter = '';
			$zexitanimationparameter = '';
		}
		
		/* get avatar and color settings */
		$i = 0;
		$zresults = $wtwconnect->query("
			select avatarpartid,
				avatarpart,
				diffusecolor,
				specularcolor,
				emissivecolor,
				ambientcolor
			from ".wtw_tableprefix."avatarcolors 
			where avatarid='".$zfoundavatarid."'
				and deleted=0
			order by avatarpart, updatedate desc, avatarpartid;");
		foreach ($zresults as $zrow) {
			$zavatarparts[$i] = array(
				'globalpartid'=>'',
				'avatarpartid'=> $zrow["avatarpartid"],
				'avatarpart'=> $zrow["avatarpart"],
				'diffusecolor'=> $zrow["diffusecolor"],
				'emissivecolor'=> $zrow["emissivecolor"],
				'specularcolor'=> $zrow["specularcolor"],
				'ambientcolor'=> $zrow["ambientcolor"]
			);
			$i += 1;
		}
		if (!empty($zstartframe) && !empty($zendframe)) {
			/* get the avatar animations (the wait idle is stored in the avatars table) */
			$zavataranimationdefs[0] = array(
				'animationind'=> 0,
				'useravataranimationid'=> '',
				'avataranimationid'=> '',
				'animationevent'=> 'onwait',
				'animationfriendlyname'=> 'Wait',
				'loadpriority'=> 100,
				'animationicon'=> '',
				'defaultspeedratio'=> 1.00,
				'speedratio'=> 1.00,
				'objectfolder'=> $zobjectfolder,
				'objectfile'=> $zobjectfile,
				'startframe'=> $zstartframe,
				'endframe'=> $zendframe,
				'animationloop'=> 1,
				'soundid'=> '',
				'soundmaxdistance'=> 100,
				'walkspeed'=> '1',
				'totalframes' => '0',
				'totalstartframe' => '0',
				'totalendframe' => '0'
			);
			$i = 1;
		} else {
			$i = 0;
		}
		/* get the avatar animations (the rest are stored in the avataranimations table) */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."avataranimations 
			where avatarid='".$zfoundavatarid."'
				and deleted=0
			order by loadpriority desc, avataranimationid;");
		foreach ($zresults as $zrow) {
			if (!empty($zrow["endframe"])) {
				$zavataranimationdefs[$i] = array(
					'animationind'=> $i,
					'useravataranimationid'=> '',
					'avataranimationid'=> $zrow["avataranimationid"],
					'animationevent'=> $zrow["animationevent"],
					'animationfriendlyname'=> addslashes($zrow["animationfriendlyname"]),
					'loadpriority'=> $zrow["loadpriority"],
					'animationicon'=> $zrow["animationicon"],
					'defaultspeedratio'=> $zrow["speedratio"],
					'speedratio'=> $zrow["speedratio"],
					'objectfolder'=> $zrow["objectfolder"],
					'objectfile'=> $zrow["objectfile"],
					'startframe'=> $zrow["startframe"],
					'endframe'=> $zrow["endframe"],
					'animationloop'=> $zrow["animationloop"],
					'soundid'=> $zrow["soundid"],
					'soundmaxdistance'=> $zrow["soundmaxdistance"],
					'walkspeed'=> '1',
					'totalframes' => '0',
					'totalstartframe' => '0',
					'totalendframe' => '0'
				);
				$i += 1;
			}
		}

	}

	$i = 0;
	$zresults = $wtwconnect->query("
		select * 
		from ".wtw_tableprefix."contentratings
		where (webid='".$zfounduseravatarid."' and not webid='' and webtype='useravatar')
			or (webid='".$zfoundavatarid."' and not webid='' and webtype='avatar')
		order by webtype desc, updatedate desc
		limit 1;");
	foreach ($zresults as $zrow) {
		$zcontentratings[$i] = array(
			'contentratingid'=> $zrow["contentratingid"],
			'webidid'=> $zrow["webid"],
			'webtype'=> $zrow["webtype"],
			'rating'=> $zrow["rating"],
			'ratingvalue'=> $zrow["ratingvalue"],
			'contentwarning'=> addslashes($zrow["contentwarning"]),
			'createdate'=> $zrow["createdate"],
			'createuserid'=> $zrow["createuserid"],
			'updatedate'=> $zrow["updatedate"],
			'updateuserid'=> $zrow["updateuserid"]
		);
		$i += 1;
	}
	
	if ($wtwconnect->hasValue($zglobalhash)) {
		/* check to see if the files and users should be included in the result set */
		$zresults = $wtwconnect->query("
			select * 
			from ".wtw_tableprefix."useravatars 
			where globalhash='".$zglobalhash."'
				and deleted=0
			limit 1;");
		foreach ($zresults as $zrow) {
			/* add files */
			$zfiles = $wtwconnect->getAvatarFilesList($zfiles, wtw_rootpath.$zrow["objectfolder"]);
			
			/* add user */
			$i = 0;
			$zresults2 = $wtwconnect->query("
				select * 
				from ".wtw_tableprefix."users 
				where userid='".$zuserid."'
					and deleted=0
				limit 1;");
			foreach ($zresults2 as $zrow2) {
				$zusers[$i] = array(
					'userid'=> $zrow2["userid"],
					'displayname'=> addslashes($zrow2["displayname"]),
					'email'=> $zrow2["email"],
					'uploadpathid'=> $zrow2["uploadpathid"]
				);
				$i += 1;
			}
		}
	}
	
	if ($wtwconnect->hasValue($zinstanceid)) {
		/* get any blocked or banned records if they exist */
		$zfoundbantable = false;
		$zresults = $wtwconnect->query("show tables like '".wtw_tableprefix."3dinternet_blockedinstances';");
		if (is_object($zresults)) {
			if ($zresults->num_rows > 0) {
				$zfoundbantable = true;
			}
		} 
		if ($zfoundbantable) {
			$i = 0;
			$j = 0;

			$zresults = $wtwconnect->query("
				select *
				from ".wtw_tableprefix."3dinternet_blockedinstances
				where (baninstanceid='".$zinstanceid."'
						or instanceid='".$zinstanceid."')
					and deleted=0;
			");
			foreach ($zresults as $zrow) {
				if ($zrow->blockchat == 1) {
					$zblockedby[$i] = array(
						'instanceid' => $zrow->instanceid,
						'baninstanceid' => $zrow->baninstanceid
					);
				}
				if ($zrow->banuser == 1) {
					$zbannedby[$i] = array(
						'instanceid' => $zrow->instanceid,
						'baninstanceid' => $zrow->baninstanceid
					);
				}
			}
		}
	}
	
	/* combine avatar settings and animations for json return dataset */
	$zavatar = array(
		'name'=> '',
		'userid'=> $zuserid,
		'userip'=> '',
		'anonymous'=>$zanonymous,
		'globaluseravatarid'=> '',
		'useravatarid'=> $zfounduseravatarid,
		'instanceid'=> $zinstanceid,
		'avatarid'=> $zfoundavatarid,
		'versionid'=> $zversionid,
		'version'=> $zversion,
		'versionorder'=> $zversionorder,
		'versiondesc'=> addslashes($zversiondesc),
		'displayname'=> addslashes($zdisplayname),
		'avatardescription'=> addslashes($zavatardescription),
		'gender'=> addslashes($zgender),
		'avatargroup'=> addslashes($zavatargroup),
		'avatargroups'=> array(),
		'privacy'=> $zprivacy,
		'scalingx'=> $zscalingx,
		'scalingy'=> $zscalingy,
		'scalingz'=> $zscalingz,
		'objectfolder'=> $zobjectfolder,
		'objectfile'=> $zobjectfile,
		'startframe'=> $zstartframe,
		'endframe'=> $zendframe,
		'position'=> array(
			'x'=> $zpositionx,
			'y'=> $zpositiony,
			'z'=> $zpositionz
		),
		'scaling'=> array(
			'x'=> $zscalingx,
			'y'=> $zscalingy,
			'z'=> $zscalingz
		),
		'rotation'=> array(
			'x'=> $zrotationx,
			'y'=> $zrotationy,
			'z'=> $zrotationz
		),
		'objects'=> array(
			'folder'=> $zobjectfolder,
			'file'=> $zobjectfile,
			'startframe'=> $zstartframe,
			'endframe'=> $zendframe
		),
		'start'=> array(
			'position'=> array(
				'x'=> 0,
				'y'=> 0,
				'z'=> 0
			),
			'rotation'=> array(
				'x'=> 0,
				'y'=> 0,
				'z'=> 0
			)
		),
		'sounds'=> array(
			'voice' => null
		),
		'avatarparts'=> $zavatarparts,
		'avataranimationdefs'=> $zavataranimationdefs,
		'animations'=> array(),
		'files'=> $zfiles,
		'users'=> $zusers,
		'contentratings'=> $zcontentratings,
		'enteranimation'=> $zenteranimation,
		'enteranimationparameter'=> $zenteranimationparameter,
		'exitanimation'=> $zexitanimation,
		'exitanimationparameter'=> $zexitanimationparameter,
		'walkspeed'=> $zwalkspeed,
		'walkanimationspeed'=> $zwalkanimationspeed,
		'turnspeed'=> $zturnspeed,
		'turnanimationspeed'=> $zturnanimationspeed,
		'checkcollisions'=> '1',
		'ispickable'=> '0',
		'moveevents'=> '',
		'parentname'=> '',
		'updated'=> '0',
		'loaded'=> '0',
		'blockedby'=> $zblockedby,
		'bannedby'=> $zbannedby
	);
	$zresponse['avatar'] = $zavatar;
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-useravatar.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: userprofile.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides basic user information for the logged in user (by userid session variable) */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/userprofile.php");
	
	$zuserid = $wtwconnect->userid;
	$zuseravatarid = $wtwconnect->getVal('useravatarid','');

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$zuser = array();

	/* get users information */
	$zresults = $wtwconnect->query("
		select u1.*,
			ua1.displayname as avatardisplayname,
			ua1.avatardescription,
			ua1.gender as avatargender
		from ".wtw_tableprefix."users u1 
			left join (select * 
				from ".wtw_tableprefix."useravatars 
				where useravatarid='".$zuseravatarid."' 
					and deleted=0) ua1
			on u1.userid=ua1.userid
		where u1.deleted=0
			and u1.userid='".$zuserid."'
		limit 1;");
	
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zdob = '';
		$zdisplayname = $zrow["displayname"];
		$zgender = $zrow["gender"];
		if ($wtwconnect->hasValue($zrow["dob"])) {
			$zdob = strtotime($zrow["dob"]);
			$zdob = date("m/d/Y", $zdob);
		}
		if ($wtwconnect->hasValue($zrow["avatardisplayname"])) {
			$zdisplayname = $zrow["avatardisplayname"];
		}
		if ((!isset($zgender) || empty($zgender)) && isset($zrow["avatargender"]) && !empty($zrow["avatargender"])) {
			$zgender = $zrow["avatargender"];
		}
		$zuser = array(
			'userid' => $zrow["userid"],
			'uploadpathid' => $zrow["uploadpathid"],
			'userimageurl' => $zrow["userimageurl"],
			'email' => $zrow["email"],
			'displayname' => $zdisplayname,
			'avatardescription' => $zrow["avatardescription"],
			'firstname' => $zrow["firstname"],
			'lastname' => $zrow["lastname"],
			'gender' => $zgender,
			'dob' => $zdob,
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"],
			'deleteddate' => $zrow["deleteddate"],
			'deleteduserid' => $zrow["deleteduserid"],
			'deleted' => $zrow["deleted"],
			'roles' => $wtwconnect->getUserRoles($zrow["userid"])
		);
	}
	echo json_encode($zuser);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-userprofile.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: users.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple users information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/users.php");
	
	/* get values from querystring or session */
	$zall = $wtwconnect->getVal('all','0');

	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zusers = array();
	
	if ($wtwconnect->hasPermission(array("admin"))) {
		$zsql = "select * 
			from ".wtw_tableprefix."users
			where deleted=0
				and pastuserid=''
			order by email, userid;";
		if ($zall == 1) {
			$zsql = "select * 
				from ".wtw_tableprefix."users
				where deleted=0
				order by email, userid;";
		}
		
		/* get users information */
		$zresults = $wtwconnect->query($zsql);
		
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			$usertokenexists = 'false';
			if ($wtwconnect->hasValue($zrow["usertoken"])) {
				$usertokenexists = 'true';
			}
			$zusers[$i] = array(
				'userid' => $zrow["userid"],
				'uploadpathid' => $zrow["uploadpathid"],
				'userimageurl' => $zrow["userimageurl"],
				'email' => $zrow["email"],
				'displayname' => $zrow["displayname"],
				'firstname' => $zrow["firstname"],
				'lastname' => $zrow["lastname"],
				'gender' => $zrow["gender"],
				'dob' => $zrow["dob"],
				'usertoken' => $usertokenexists,
				'createdate' => $zrow["createdate"],
				'createuserid' => $zrow["createuserid"],
				'updatedate' => $zrow["updatedate"],
				'updateuserid' => $zrow["updateuserid"],
				'deleteddate' => $zrow["deleteddate"],
				'deleteduserid' => $zrow["deleteduserid"],
				'deleted' => $zrow["deleted"],
				'roles' => $wtwconnect->getUserRoles($zrow["userid"])
				);
			$i += 1;
		}
	}
	echo json_encode($zusers);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-users.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webalias.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web aliases information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/webalias.php");
	
	$zwebaliasid = $wtwconnect->getVal('webaliasid','');
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	
	$zresponse = array();
	if ($wtwconnect->hasValue($zwebaliasid) && ($wtwconnect->isUserInRole("Admin") || $wtwconnect->isUserInRole("Host"))) {
		/* get web aliases for a user */
		$zresults = array();
		if ($wtwconnect->isUserInRole("Admin")) {
			$zresults = $wtwconnect->query("
				select w1.*,
					c1.communityname,
					b1.buildingname,
					t1.thingname,
					c1.snapshotid as communitysnapshotid,
					b1.snapshotid as buildingsnapshotid,
                    t1.snapshotid as thingsnapshotid,
					case when c1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=c1.snapshotid limit 1)
						end as communitysnapshoturl,
					case when b1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=b1.snapshotid limit 1)
						end as buildingsnapshoturl,
					case when t1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=t1.snapshotid limit 1)
						end as thingsnapshoturl,
					case when w1.siteiconid = '' then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=w1.siteiconid limit 1)
						end as siteiconpath
				from ".wtw_tableprefix."webaliases w1
					left join ".wtw_tableprefix."communities c1
						on w1.communityid=c1.communityid
					left join ".wtw_tableprefix."buildings b1
						on w1.buildingid=b1.buildingid
					left join ".wtw_tableprefix."things t1
						on w1.thingid=t1.thingid
				where w1.deleted=0
					and w1.webaliasid='".$zwebaliasid."'
				order by 
					w1.hostuserid,
					w1.domainname,
					w1.communitypublishname,
					w1.buildingpublishname,
					w1.thingpublishname,
					w1.communityid,
					w1.buildingid,
					w1.thingid,
					w1.webaliasid;");
		} else {
			$zresults = $wtwconnect->query("
				select w1.*,
					c1.communityname,
					b1.buildingname,
					t1.thingname,
					c1.snapshotid as communitysnapshotid,
					b1.snapshotid as buildingsnapshotid,
                    t1.snapshotid as thingsnapshotid,
					case when c1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=c1.snapshotid limit 1)
						end as communitysnapshoturl,
					case when b1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=b1.snapshotid limit 1)
						end as buildingsnapshoturl,
					case when t1.snapshotid is null then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=t1.snapshotid limit 1)
						end as thingsnapshoturl,
					case when w1.siteiconid = '' then ''
						else (select filepath 
							from ".wtw_tableprefix."uploads 
							where uploadid=w1.siteiconid limit 1)
						end as siteiconpath
				from ".wtw_tableprefix."webaliases w1
					left join ".wtw_tableprefix."communities c1
						on w1.communityid=c1.communityid
					left join ".wtw_tableprefix."buildings b1
						on w1.buildingid=b1.buildingid
					left join ".wtw_tableprefix."things t1
						on w1.thingid=t1.thingid
				where w1.deleted=0
					and w1.webaliasid='".$zwebaliasid."'
					and w1.hostuserid='".$zhostuserid."'
					and not w1.hostuserid=''
				order by 
					w1.hostuserid,
					w1.domainname,
					w1.communitypublishname,
					w1.buildingpublishname,
					w1.thingpublishname,
					w1.communityid,
					w1.buildingid,
					w1.thingid,
					w1.webaliasid;");
		}
		echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
		
		$i = 0;
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			$webalias = array(
				'serverfranchiseid' => '',
				'webaliasid' => $zrow["webaliasid"],
				'domainname' => $zrow["domainname"],
				'webalias' => $zrow["webalias"],
				'sitename' => $zrow["sitename"],
				'sitedescription' => $zrow["sitedescription"],
				'siteiconid' => $zrow["siteiconid"],
				'siteiconpath' => $zrow["siteiconpath"],
				'communityid' => $zrow["communityid"],
				'communitypublishname' => $zrow["communitypublishname"],
				'communityname' => $zrow["communityname"],
				'buildingid' => $zrow["buildingid"],
				'buildingpublishname' => $zrow["buildingpublishname"],
				'buildingname' => $zrow["buildingname"],
				'thingid' => $zrow["thingid"],
				'thingpublishname' => $zrow["thingpublishname"],
				'thingname' => $zrow["thingname"],
				'forcehttps' => $zrow["forcehttps"],
				'franchise' => $zrow["franchise"],
				'franchiseid' => $zrow["franchiseid"],
				'communitysnapshotid' => $zrow["communitysnapshotid"],
				'buildingsnapshotid' => $zrow["buildingsnapshotid"],
				'thingsnapshotid' => $zrow["thingsnapshotid"],
				'communitysnapshoturl' => $zrow["communitysnapshoturl"],
				'buildingsnapshoturl' => $zrow["buildingsnapshoturl"],
				'thingsnapshoturl' => $zrow["thingsnapshoturl"],
				'createdate' => $zrow["createdate"],
				'createuserid' => $zrow["createuserid"],
				'updatedate' => $zrow["updatedate"],
				'updateuseride' => $zrow["updateuserid"]);
			$zresponse[$i] = $webalias;
			$i += 1;
		}
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-webalias.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webaliases.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple web aliases information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/webaliases.php");
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	
	$zresults = array();
	
	/* get web aliases for a user */
	if ($wtwconnect->isUserInRole("Admin") || $wtwconnect->isUserInRole("Developer")) {
		$zresults = $wtwconnect->query("
			select w1.*,
				c1.communityname,
				b1.buildingname,
				t1.thingname,
				c1.snapshotid as communitysnapshotid,
				b1.snapshotid as buildingsnapshotid,
				t1.snapshotid as thingsnapshotid,
				case when c1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=c1.snapshotid limit 1)
					end as communitysnapshoturl,
				case when b1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=b1.snapshotid limit 1)
					end as buildingsnapshoturl,
				case when t1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=t1.snapshotid limit 1)
					end as thingsnapshoturl,
				case when w1.siteiconid = '' then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=w1.siteiconid limit 1)
					end as siteiconpath
			from ".wtw_tableprefix."webaliases w1
				left join ".wtw_tableprefix."communities c1
					on w1.communityid=c1.communityid
				left join ".wtw_tableprefix."buildings b1
					on w1.buildingid=b1.buildingid
				left join ".wtw_tableprefix."things t1
					on w1.thingid=t1.thingid
			where w1.deleted=0
			order by 
				w1.hostuserid desc,
				w1.domainname,
				w1.communitypublishname,
				w1.buildingpublishname,
				w1.thingpublishname,
				w1.communityid,
				w1.buildingid,
				w1.thingid,
				w1.webaliasid;");
	} else if ($wtwconnect->isUserInRole("Host")) {
		$zresults = $wtwconnect->query("
			select w1.*,
				c1.communityname,
				b1.buildingname,
				t1.thingname,
				c1.snapshotid as communitysnapshotid,
				b1.snapshotid as buildingsnapshotid,
				t1.snapshotid as thingsnapshotid,
				case when c1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=c1.snapshotid limit 1)
					end as communitysnapshoturl,
				case when b1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=b1.snapshotid limit 1)
					end as buildingsnapshoturl,
				case when t1.snapshotid is null then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=t1.snapshotid limit 1)
					end as thingsnapshoturl,
				case when w1.siteiconid = '' then ''
					else (select filepath 
						from ".wtw_tableprefix."uploads 
						where uploadid=w1.siteiconid limit 1)
					end as siteiconpath
			from ".wtw_tableprefix."webaliases w1
				left join ".wtw_tableprefix."communities c1
					on w1.communityid=c1.communityid
				left join ".wtw_tableprefix."buildings b1
					on w1.buildingid=b1.buildingid
				left join ".wtw_tableprefix."things t1
					on w1.thingid=t1.thingid
			where w1.deleted=0
				and w1.hostuserid='".$zhostuserid."'
				and not w1.hostuserid=''
			order by 
				w1.hostuserid,
				w1.domainname,
				w1.communitypublishname,
				w1.buildingpublishname,
				w1.thingpublishname,
				w1.communityid,
				w1.buildingid,
				w1.thingid,
				w1.webaliasid;");
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$webalias = array(
			'serverfranchiseid' => '',
			'webaliasid' => $zrow["webaliasid"],
			'domainname' => $zrow["domainname"],
			'webalias' => $zrow["webalias"],
			'sitename' => $zrow["sitename"],
			'sitedescription' => $zrow["sitedescription"],
			'siteiconid' => $zrow["siteiconid"],
			'siteiconpath' => $zrow["siteiconpath"],
			'communityid' => $zrow["communityid"],
			'communitypublishname' => $zrow["communitypublishname"],
			'communityname' => $zrow["communityname"],
			'buildingid' => $zrow["buildingid"],
			'buildingpublishname' => $zrow["buildingpublishname"],
			'buildingname' => $zrow["buildingname"],
			'thingid' => $zrow["thingid"],
			'thingpublishname' => $zrow["thingpublishname"],
			'thingname' => $zrow["thingname"],
			'forcehttps' => $zrow["forcehttps"],
			'franchise' => $zrow["franchise"],
			'franchiseid' => $zrow["franchiseid"],
			'communitysnapshotid' => $zrow["communitysnapshotid"],
			'buildingsnapshotid' => $zrow["buildingsnapshotid"],
			'thingsnapshotid' => $zrow["thingsnapshotid"],
			'communitysnapshoturl' => $zrow["communitysnapshoturl"],
			'buildingsnapshoturl' => $zrow["buildingsnapshoturl"],
			'thingsnapshoturl' => $zrow["thingsnapshoturl"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"]);
		$zresponse[$i] = $webalias;
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-webaliases.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webdomain.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web domain information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/webdomain.php");
	
	$zwebdomainid = $wtwconnect->getVal('webdomainid','');
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	
	$zresponse = array();
	if ($wtwconnect->hasValue($zwebdomainid) && ($wtwconnect->isUserInRole("Admin") || $wtwconnect->isUserInRole("Host"))) {
		/* get web aliases for a user */
		$zresults = array();
		if ($wtwconnect->isUserInRole("Admin")) {
			$zresults = $wtwconnect->query("
			select w1.*
			from ".wtw_tableprefix."webdomains w1
			where w1.deleted=0
				and webdomainid='".$zwebdomainid."'
			order by 
				w1.hostuserid desc,
				w1.domainname,
				w1.webdomainid;");
		} else {
			$zresults = $wtwconnect->query("
			select w1.*
			from ".wtw_tableprefix."webdomains w1
			where w1.deleted=0
				and webdomainid='".$zwebdomainid."'
				and hostuserid='".$zhostuserid."'
			order by 
				w1.hostuserid,
				w1.domainname,
				w1.webdomainid;");
		}
		echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
		
		$i = 0;
		/* format json return dataset */
		foreach ($zresults as $zrow) {
			$zwebdomain = array(
				'serverfranchiseid' => '',
				'webdomainid' => $zrow["webdomainid"],
				'hostuserid' => $zrow["hostuserid"],
				'domainname' => $zrow["domainname"],
				'forcehttps' => $zrow["forcehttps"],
				'allowhosting' => $zrow["allowhosting"],
				'startdate' => $zrow["startdate"],
				'expiredate' => $zrow["expiredate"],
				'hostprice' => $zrow["hostprice"],
				'hostdays' => $zrow["hostdays"],
				'createdate' => $zrow["createdate"],
				'createuserid' => $zrow["createuserid"],
				'updatedate' => $zrow["updatedate"],
				'updateuserid' => $zrow["updateuserid"]);
			$zresponse[$i] = $zwebdomain;
			$i += 1;
		}
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-webdomain.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webdomains.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple web domains information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/webdomains.php");
	$zhostuserid = '';
	if ($wtwconnect->isUserInRole("Host") && $wtwconnect->isUserInRole("Admin") == false) {
		$zhostuserid = $wtwconnect->userid;
	}
	
	$zresults = array();
	
	/* get web domains for a user */
	if ($wtwconnect->isUserInRole("Admin")) {
		$zresults = $wtwconnect->query("
			select w1.*
			from ".wtw_tableprefix."webdomains w1
			where w1.deleted=0
			order by 
				w1.hostuserid desc,
				w1.domainname,
				w1.webdomainid;");
	} else if ($wtwconnect->isUserInRole("Host")) {
		$zresults = $wtwconnect->query("
			select w1.*
			from ".wtw_tableprefix."webdomains w1
			where w1.deleted=0
				and (hostuserid='".$zhostuserid."'
				or (hostuserid='' and allowhosting=1))
			order by 
				w1.hostuserid,
				w1.domainname,
				w1.webdomainid;");
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zwebdomain = array(
			'serverfranchiseid' => '',
			'webdomainid' => $zrow["webdomainid"],
			'hostuserid' => $zrow["hostuserid"],
			'domainname' => $zrow["domainname"],
			'forcehttps' => $zrow["forcehttps"],
			'allowhosting' => $zrow["allowhosting"],
			'startdate' => $zrow["startdate"],
			'expiredate' => $zrow["expiredate"],
			'hostprice' => $zrow["hostprice"],
			'hostdays' => $zrow["hostdays"],
			'createdate' => $zrow["createdate"],
			'createuserid' => $zrow["createuserid"],
			'updatedate' => $zrow["updatedate"],
			'updateuserid' => $zrow["updateuserid"]);
		$zresponse[$i] = $zwebdomain;
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-webdomains.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webnamecheck.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web aliases information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/webnamecheck.php");
	
	$zwebname = $wtwconnect->getVal('webname','');
	$zwebname = strtolower($wtwconnect->decode64($zwebname));
	
	$zresponse = array(
		'serror'=>'',
		'available'=>'0',
		'webname'=>$zwebname
	);
	
	if ($wtwconnect->hasValue($zwebname)) {
		
		/* reserved words can not be any part of the webname - you can add your own reserved words */
		$zreserved = array('wtw','walktheweb','http3d','https3d');

		$zwebtest = str_replace("_","",str_replace(".","",str_replace("-","",$zwebname)));

		$zfound = '0';
		foreach ($zreserved as $zword) {
			if (strpos($zwebtest, $zword) !== false) {
				 $zfound = '1';
			}
		}

		if ($zfound == '1') {
			$zresponse = array(
				'serror'=>'Webname Contains Reserved Word',
				'available'=>'0',
				'webname'=>$zwebname
			);
		} else {
			/* check if web alias is already in use */
			$zresults = $wtwconnect->query("
				select w1.*
				from ".wtw_tableprefix."webaliases w1
				where w1.deleted=0
					and (w1.communitypublishname='".$zwebname."'
						or w1.buildingpublishname='".$zwebname."');");
			if (count($zresults) > 0) {
				$zresponse = array(
					'serror'=>'Webname Already in Use',
					'available'=>'0',
					'webname'=>$zwebname
				);
			} else {
				$zresponse = array(
					'serror'=>'',
					'available'=>'1',
					'webname'=>$zwebname
				);
			}
		}
	}
	echo $wtwconnect->addConnectHeader('*');
	
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-webnamecheck.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: webs.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides multiple 3D Communities information */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/communities.php");

	/* get values from querystring or session */
	$zuserid = $wtwconnect->userid;
	$zfilter = $wtwconnect->getVal('filter','mine');

	/* check user for global roles with access */
	$hasaccess = false;
	if ($zfilter == 'all') {
		$zroles = $wtwconnect->getUserRoles($zuserid);
		foreach ($zroles as $zrole) {
			if (strtolower($zrole['rolename']) == 'admin' || strtolower($zrole['rolename']) == 'architect' || strtolower($zrole['rolename']) == 'developer' || strtolower($zrole['rolename']) == 'graphics artist') {
				$hasaccess = true;
			}
		}
	}
	/* select webs by userid */
	$zresults = array();


	$zcontentpath = '';
	if (defined('wtw_contentpath')) {
		$zcontentpath = wtw_contentpath;
	}
	$zsearch = '';
	$zwebtype = '';
	$zwebtypes = '';
	$zlist = '';
	$zversionid = '';
	$zsorder = '';
	$zpermissions = "";
	if(isset($_GET['search']) && !empty($_GET['search'])) {
		$zsearch = $_GET['search'];
	}
	if(isset($_GET['webtype']) && !empty($_GET['webtype'])) {
		$zwebtype = $_GET['webtype'];
	}
	if(isset($_GET['list']) && !empty($_GET['list'])) {
		$zlist = $_GET['list'];
	}
	if(isset($_GET['versionid']) && !empty($_GET['versionid'])) {
		$zversionid = $_GET['versionid'];
	}
	
	$zsearch = str_replace(" ","%",$zsearch);
	
	switch ($zwebtype) {
		case "community":
			$zwebtypes = "communities";
			break;
		case "building":
			$zwebtypes = "buildings";
			break;
		case "thing":
			$zwebtypes = "things";
			break;
		case "avatar":
			$zwebtypes = "avatars";
			break;
		case "plugin":
			$zwebtypes = "plugins";
			break;
	}
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);
	
	if ($zwebtype == 'avatar') {
		$zpermissions = " and user1.userid='".$zuserid."' ";
		if ($zfilter == 'all' && $hasaccess) {
			$zpermissions = '';
		}
		if (isset($zlist) && !empty($zlist)) {
			switch ($zlist) {
				case "latest":
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.displayname, t1.templatename, t1.".$zwebtype."id";
					break;
				case "version":
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.displayname, t1.templatename, t1.".$zwebtype."id";
					break;
				default:
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.displayname, t1.templatename, t1.".$zwebtype."id";
					break;
			}
		} else {
			$zsorder = "t1.displayname, t1.templatename, t1.createdate desc, t1.updatedate desc, t1.".$zwebtype."id";
		}
		if ($zlist == 'version' && isset($zversionid) && !empty($zversionid)) {
			$zsql = "select distinct t1.*, 
						t1.".$zwebtype."id as webid,
						t1.displayname as webname,
						u1.filepath as imageurl,
						user1.displayname as username,
						user1.userid,
						user1.email,
						user2.displayname as createusername,
						user2.email as createemail,
						t1.displayname as avatarname,
						m1.versionmax
					from wtw_".$zwebtypes." t1
						left join wtw_uploads u1
						on t1.snapshotid = u1.uploadid
						left join wtw_users user1
						on t1.updateuserid = user1.userid
						left join wtw_users user2
						on t1.createuserid = user2.userid
						left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
						on t1.versionid=m1.versionid
					where t1.deleted=0 
						and t1.versionid='".$zversionid."'
						".$zpermissions."
					order by ".$zsorder.";";
		} else if (!empty($zsearch) && isset($zsearch) && $zsearch != '*' && $zsearch != '%') {
			$zsql = "select distinct t1.*, 
					t1.".$zwebtype."id as webid,
					t1.displayname as webname,
					u1.filepath as imageurl,
					user1.displayname as username,
					user1.userid,
					user1.email,
					user2.displayname as createusername,
					user2.email as createemail,
                    t1.displayname as avatarname,
                    m1.versionmax
				from wtw_".$zwebtypes." t1
					left join wtw_uploads u1
					on t1.snapshotid = u1.uploadid
					left join wtw_users user1
					on t1.updateuserid = user1.userid
					left join wtw_users user2
					on t1.createuserid = user2.userid
					left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
                    on t1.versionid=m1.versionid
				where t1.deleted=0 
					and (t1.displayname like '%".$zsearch."%'
					or t1.".$zwebtype."id like '%".$zsearch."%'
					or t1.templatename like '%".$zsearch."%'
					or t1.tags like '%".$zsearch."%'
					or t1.versiondesc like '%".$zsearch."%'
					or t1.versionid like '%".$zsearch."%'
					or t1.description like '%".$zsearch."%')
					and t1.deleted=0
					".$zpermissions."
				order by ".$zsorder.";";
		} else {
			$zsql = "select distinct t1.*, 
						t1.".$zwebtype."id as webid,
						t1.displayname as webname,
						u1.filepath as imageurl,
						user1.displayname as username,
						user1.userid,
						user1.email,
						user2.displayname as createusername,
						user2.email as createemail,
						t1.displayname as avatarname,
						m1.versionmax
					from wtw_".$zwebtypes." t1
						left join wtw_uploads u1
						on t1.snapshotid = u1.uploadid
						left join wtw_users user1
						on t1.updateuserid = user1.userid
						left join wtw_users user2
						on t1.createuserid = user2.userid
						left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
						on t1.versionid=m1.versionid
					where t1.deleted=0 
						".$zpermissions."
					order by ".$zsorder.";";
		}
		
	} else if ($zwebtype == 'plugin') {
		$zpermissions = " and user1.userid='".$zuserid."' ";
		if ($zfilter == 'all' && $hasaccess) {
			$zpermissions = '';
		}
		if (isset($zlist) && !empty($zlist)) {
			switch ($zlist) {
				case "latest":
					$zsorder = "t1.createdate desc, t1.templatename, t1.pluginname, t1.pluginid";
					break;
				default:
					$zsorder = "t1.createdate desc, t1.templatename, t1.pluginname, t1.pluginid";
					break;
			}
		} else {
			$zsorder = "t1.templatename, t1.pluginname, t1.versionorder desc, t1.createdate desc, t1.pluginid";
		}
		$zsql = "select distinct t1.*, 
					t1.".$zwebtype."id as webid,
					t1.".$zwebtype."name as webname,
					user1.displayname as username,
					user1.email,
					user2.displayname as createusername,
					user2.email as createemail,
					'' as snapshotid
				from wtw_".$zwebtypes." t1
					left join wtw_users user1
					on t1.userid = user1.userid
					left join wtw_users user2
					on t1.createuserid = user2.userid
				where t1.deleted=0 
					".$zpermissions."
				order by ".$zsorder.";";

		if (!empty($zsearch) && isset($zsearch) && $zsearch != '*' && $zsearch != '%') {
			$zsql = "select distinct t1.*,
						t1.".$zwebtype."id as webid,
						t1.".$zwebtype."name as webname,
						user1.displayname as username,
						user1.email,
						user2.displayname as createusername,
						user2.email as createemail,
						'' as snapshotid
					from wtw_".$zwebtypes." t1
						left join wtw_users user1
						on t1.userid = user1.userid
						left join wtw_users user2
						on t1.createuserid = user2.userid
					where t1.deleted=0 
						and (t1.templatename like '%".$zsearch."%'
						or t1.pluginname like '%".$zsearch."%'
						or t1.pluginid like '%".$zsearch."%'
						or t1.description like '%".$zsearch."%')
						".$zpermissions."
					order by ".$zsorder.";";
		}		
	} else {
		$zpermissions = " and t1.userid='".$zuserid."' ";
		if ($zfilter == 'all' && $hasaccess) {
			$zpermissions = '';
		}
		if (isset($zlist) && !empty($zlist)) {
			switch ($zlist) {
				case "latest":
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
					break;
				case "version":
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
					break;
				default:
					$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
					break;
			}
		} else {
			$zsorder = "t1.".$zwebtype."name, t1.createdate desc, t1.".$zwebtype."id";
		}

		if ($zlist == 'version' && isset($zversionid) && !empty($zversionid)) {

			$zsql = "select distinct t1.*, 
						t1.".$zwebtype."id as webid,
						t1.".$zwebtype."name as webname,
						u1.filepath as imageurl,
						user1.displayname as username,
						user1.email,
						user2.displayname as createusername,
						user2.email as createemail
						m1.versionmax
					from wtw_".$zwebtypes." t1
						left join wtw_uploads u1
						on t1.snapshotid = u1.uploadid
						left join wtw_users user1
						on t1.userid = user1.userid
						left join wtw_users user2
						on t1.createuserid = user2.userid
						left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
						on t1.versionid=m1.versionid
					where t1.deleted=0 
						and t1.versionid='".$zversionid."'
						".$zpermissions."
					order by ".$zsorder.";";
		
		} else if (!empty($zsearch) && isset($zsearch) && $zsearch != '*' && $zsearch != '%') {
			if (isset($zlist) && !empty($zlist)) {
				switch ($zlist) {
					case "latest":
						$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
						break;
					default:
						$zsorder = "t1.createdate desc, t1.versionid, t1.updatedate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
						break;
				}
			} else {
				$zsorder = "t1.createdate desc, t1.".$zwebtype."name, t1.templatename, t1.".$zwebtype."id";
			}
			$zsql = "select distinct t1.*,
					t1.".$zwebtype."id as webid,
					t1.".$zwebtype."name as webname,
					u1.filepath as imageurl,
					user1.displayname as username,
					user1.email,
					user2.displayname as createusername,
					user2.email as createemail,
                    m1.versionmax
				from wtw_".$zwebtypes." t1
					left join wtw_uploads u1
					on t1.snapshotid = u1.uploadid
					left join wtw_users user1
					on t1.userid = user1.userid
					left join wtw_users user2
					on t1.createuserid = user2.userid
					left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
                    on t1.versionid=m1.versionid
				where (t1.".$zwebtype."name like '%".$zsearch."%'
					or t1.".$zwebtype."id like '%".$zsearch."%'
					or t1.templatename like '%".$zsearch."%'
					or t1.tags like '%".$zsearch."%'
					or t1.versiondesc like '%".$zsearch."%'
					or t1.versionid like '%".$zsearch."%'
					or t1.description like '%".$zsearch."%')
					and t1.deleted=0
					".$zpermissions."
				order by ".$zsorder.";";
		} else {
			$zsql = "select distinct t1.*, 
						t1.".$zwebtype."id as webid,
						t1.".$zwebtype."name as webname,
						u1.filepath as imageurl,
						user1.displayname as username,
						user1.email,
						user2.displayname as createusername,
						user2.email as createemail,
						m1.versionmax
					from wtw_".$zwebtypes." t1
						left join wtw_uploads u1
						on t1.snapshotid = u1.uploadid
						left join wtw_users user1
						on t1.userid = user1.userid
						left join wtw_users user2
						on t1.createuserid = user2.userid
						left join (select versionid, max(versionorder) as versionmax from wtw_".$zwebtypes." group by versionid) m1
						on t1.versionid=m1.versionid
					where t1.deleted=0
						".$zpermissions."
					order by ".$zsorder.";";
		}
	}

	$i = 0;
	$zresponse = array();
	
	$zresults = $wtwconnect->query($zsql);

	foreach ($zresults as $zrow) {
		$zimageurl = $zrow["imageurl"];
		if ($zwebtype == 'avatar') {
			$zimageurl = "/content/uploads/avatars/".$zrow[$zwebtype."id"]."/snapshots/defaultavatar.png";
			if (file_exists($wtwconnect->rootpath.$zimageurl) == false) {
				$zimageurl = "https://3dnet.walktheweb.com/content/uploads/avatars/".$zrow[$zwebtype."id"]."/snapshots/defaultavatar.png";
			}
		}
		/* get size of web */
		$zdirsize = 0;
		$zfileind = 0;
		$zfilelist = array();
		$zwebpath = $zcontentpath.'/uploads/'.$zwebtypes.'/'.$zrow[$zwebtype."id"];
		if (file_exists($zwebpath)) {
			$zdirsize = $wtwconnect->dirSize($zwebpath);
			$zfilelist = $wtwconnect->getFileList($zwebpath);
			$zfileind = count($zfilelist) - 1;
		}
		
		/* check for child webs */
		$zresults2 = $wtwconnect->query("
			select childwebtype, childwebid
			from wtw_connectinggrids
			where deleted=0
				and parentwebtype='".$zwebtype."'
				and parentwebid='".$zrow[$zwebtype."id"]."'
			group by childwebtype, childwebid;
		");
		foreach ($zresults2 as $zrow2) {
			$zwebtypes2 = 'communities';
			switch ($zrow2["childwebtype"]) {
				case 'building':
					$zwebtypes2 = 'buildings';
					break;
				case 'thing':
					$zwebtypes2 = 'things';
					break;
			}
			$zfilelist2 = array();
			$zwebpath2 = $zcontentpath.'/uploads/'.$zwebtypes2.'/'.$zrow2["childwebid"];
			if (file_exists($zwebpath2)) {
				$zdirsize += $wtwconnect->dirSize($zwebpath2);
				$zfilelist2 = $wtwconnect->getFileList($zwebpath2);
			}
			foreach ($zfilelist2 as $zfile2) {
				$zfilelist[$zfileind] = array(
					'filename' => $zfile2["filename"],
					'filepath' => $zfile2["filepath"],
					'filesize' => $zfile2["filesize"],
					'downloaded' => '0'
				);
				$zfileind += 1;
			}
			
			/* check for child webs of child webs (community with building with things) */
			$zresults3 = $wtwconnect->query("
				select childwebtype, childwebid
				from wtw_connectinggrids
				where deleted=0
					and parentwebtype='".$zrow2["childwebtype"]."'
					and parentwebid='".$zrow2["childwebid"]."'
				group by childwebtype, childwebid;
			");
			foreach ($zresults3 as $zrow3) {
				$zwebtypes3 = 'communities';
				switch ($zrow3["childwebtype"]) {
					case 'building':
						$zwebtypes3 = 'buildings';
						break;
					case 'thing':
						$zwebtypes3 = 'things';
						break;
				}
				$zfilelist3 = array();
				$zwebpath3 = $zcontentpath.'/uploads/'.$zwebtypes3.'/'.$zrow3["childwebid"];
				if (file_exists($zwebpath3)) {
					$zdirsize += $wtwconnect->dirSize($zwebpath3);
					$zfilelist3 = $wtwconnect->getFileList($zwebpath3);
				}
				foreach ($zfilelist3 as $zfile3) {
					$zfilelist[$zfileind] = array(
						'filename' => $zfile3["filename"],
						'filepath' => $zfile3["filepath"],
						'filesize' => $zfile3["filesize"],
						'downloaded' => '0'
					);
					$zfileind += 1;
				}
			}
		}
		
		$zresponse[$i] = array(
			$zwebtype.'id' => $zrow[$zwebtype."id"],
			$zwebtype.'name' => $zrow[$zwebtype."name"],
			'webid' => $zrow["webid"],
			'webname' => $zrow["webname"],
			'templatename' => $zrow["templatename"],
			'description' => $zrow["description"],
			'versionid' => $zrow["versionid"],
			'version' => $zrow["version"],
			'versionorder' => $zrow["versionorder"],
			'versiondesc' => $zrow["versiondesc"],
			'versionmax' => $zrow["versionmax"],
			'dirsize' => $zdirsize,
			'filecount' => count($zfilelist),
			'filelist' => $zfilelist,
			'tags' => $zrow["tags"],
			'snapshotid' => $zrow["snapshotid"],
			'imageurl' => $zimageurl,
			'userid' => $zrow["userid"],
			'createemail' => $zrow["createemail"],
			'email' => $zrow["email"],
			'createdisplayname' => $zrow["createusername"],
			'displayname' => $zrow["username"],
			'createdate' => $zrow["createdate"],
			'updatedate' => $zrow["updatedate"]
		);
		$i += 1;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
    $wtwconnect->serror("connect-webs.php: ".$e->getMessage());
}
?>

----------------------
----------------------
File: websitems.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides thing mold information list to recover a deleted item */
require_once('../core/functions/class_wtwconnect.php');
global $wtwconnect;
try {
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/websitems.php");
	
	/* get values from querystring or session */
	$zwebid = $wtwconnect->getVal('webid','');
	$zwebtype = $wtwconnect->getVal('webtype','');

	$zwebtypes = 'things';
	switch ($zwebtype) {
		case 'building':
			$zwebtypes = 'buildings';
			break;
		case 'community':
			$zwebtypes = 'communities';
			break;
	}

	/* get thing molds that have been deleted */
	$zresults = $wtwconnect->query("
		select 
			'Molds' as category,
			'1' as sorder,
			tm1.".$zwebtype."moldid as itemid,
			tm1.".$zwebtype."id as webid,
			tm1.shape as itemtype,
			case when tm1.shape = 'babylonfile' then 
					(select uo1.objectfile from ".wtw_tableprefix."uploadobjects uo1 where uo1.uploadobjectid=tm1.uploadobjectid)
				else tm1.shape
				end as webname
		from ".wtw_tableprefix.$zwebtype."molds tm1
		where tm1.".$zwebtype."id='".$zwebid."'
			and tm1.deleted=0

		union

		select 
			'Action Zones' as category,
			'3' as sorder,
			actionzoneid as itemid,
			a1.".$zwebtype."id as webid,
			actionzonetype as itemtype,
			actionzonename as webname
		from ".wtw_tableprefix."actionzones a1
		where a1.".$zwebtype."id='".$zwebid."'
			and a1.deleted=0

		union

		select 
			'3D Webs' as category,
			'2' as sorder,
			c1.connectinggridid as itemid,
			c1.childwebid as webid,
			c1.childwebtype as itemtype,
			case when c1.childwebtype = 'building' then 
					b1.buildingname
				else t1.thingname
				end as webname
		from ".wtw_tableprefix."connectinggrids c1
			left join ".wtw_tableprefix."things t1
				on c1.childwebid=t1.thingid
				and c1.childwebtype='thing'
			left join ".wtw_tableprefix."buildings b1
				on c1.childwebid=b1.buildingid
				and c1.childwebtype='building'
		where c1.parentwebid='".$zwebid."'
			and c1.deleted=0

		order by sorder, itemtype, webname;");
	
	echo $wtwconnect->addConnectHeader($wtwconnect->domainname);

	$i = 0;
	$zresponse = array();
	/* format json return dataset */
	foreach ($zresults as $zrow) {
		$zresponse[$i] = array(
			'category'=> $zrow["category"], 
			'itemid'=> $zrow["itemid"],
			'webid'=> $zrow["webid"],
			'itemtype'=> $zrow["itemtype"],
			'webname'=> $zrow["webname"]);
		$i += 1;
	}
	echo json_encode($zresponse);
} catch (Exception $e) {
	$wtwconnect->serror("connect-websitems.php=".$e->getMessage());
}
?>

----------------------
----------------------
File: wordpress.php
Content:
<?php
/* these /connect files are designed to extend data to other servers - like having your 3D Building in their 3D Community Scene */
/* permissions are required for access to some data */
/* this connect file provides web aliases information */
require_once('../core/functions/class_wtwconnect.php');
require_once('../content/plugins/wtw-3dinternet/functions/class_downloads.php');
global $wtwconnect;
global $wtw_3dinternet_downloads;

try {
	echo $wtwconnect->addConnectHeader('*');
	/* google analytics tracking (if defined in wtw_config.php) */
	$wtwconnect->trackPageView($wtwconnect->domainurl."/connect/wordpress.php");
	
	$zfunction = $wtwconnect->getPost('function','');
	$zwpinstanceid = $wtwconnect->getPost('wpinstanceid','');
	$zwebsiteurl = $wtwconnect->getPost('websiteurl','');
	$zbuildingid = $wtwconnect->getPost('buildingid','');
	$zcommunityid = $wtwconnect->getPost('communityid','');
	$zwebid = $wtwconnect->getPost('webid','');
	$zwebtype = $wtwconnect->getPost('webtype','');
	$zusertoken = $wtwconnect->getPost('usertoken','');
	$zwtwusertoken = $wtwconnect->getPost('wtwusertoken','');
	$zwtwemail = $wtwconnect->getPost('wtwemail','');
	$zwtwuserid = $wtwconnect->getPost('userid','');
	$zhosturl = $wtwconnect->getPost('hosturl','');
	$zwtwurl = $wtwconnect->getPost('wtwurl','');
	$zwebname = $wtwconnect->getPost('webname','');
	$zwtwstorename = $wtwconnect->getPost('wtwstorename','');
	$zwtwkey = $wtwconnect->getPost('wtwkey','');
	$zwtwsecret = $wtwconnect->getPost('wtwsecret','');
	$zwookey = $wtwconnect->getPost('wookey','');
	$zwoosecret = $wtwconnect->getPost('woosecret','');
	$zstoreurl = $wtwconnect->getPost('storeurl','');
	$zstorecarturl = $wtwconnect->getPost('storecarturl','');
	$zstoreproducturl = $wtwconnect->getPost('storeproducturl','');
	$zstoreapiurl = $wtwconnect->getPost('storeapiurl','');
	$ziframes = $wtwconnect->getPost('iframes','');

	$zauthenticationok = false;
	$zwebnameok = false;
	$zuserid = $wtwconnect->userid;
	$znewcommunityid = '';
	$znewbuildingid = '';
	$zdomainname = '';
	$zforcehttps = '1';
	$serror = '';
	
	$zwpinstanceid = $wtwconnect->decode64($zwpinstanceid);
	$zwebsiteurl = $wtwconnect->decode64($zwebsiteurl);
	$zbuildingid = $wtwconnect->decode64($zbuildingid);
	$zcommunityid = $wtwconnect->decode64($zcommunityid);
	$zwtwemail = strtolower($wtwconnect->decode64($zwtwemail));
	$zwtwuserid = $wtwconnect->decode64($zwtwuserid);
	$zhosturl = $wtwconnect->decode64($zhosturl);
	$zwtwurl = $wtwconnect->decode64($zwtwurl);
	$zwebname = strtolower($wtwconnect->decode64($zwebname));
	$zwtwstorename = $wtwconnect->decode64($zwtwstorename);
	$zwtwsecret = $wtwconnect->decode64($zwtwsecret);
	$zwookey = $wtwconnect->decode64($zwookey);
	$zwoosecret = $wtwconnect->decode64($zwoosecret);
	$zstoreurl = $wtwconnect->decode64($zstoreurl);
	$zstorecarturl = $wtwconnect->decode64($zstorecarturl);
	$zstoreproducturl = $wtwconnect->decode64($zstoreproducturl);
	$zstoreapiurl = $wtwconnect->decode64($zstoreapiurl);
	$ziframes = $wtwconnect->decode64($ziframes);
	$zhostuserid = '';

	try {
		$zparse = parse_url($zhosturl);
		$zdomainname = $zparse['host'];
		if (strpos(strtolower($zhosturl), 'http://') !== false) {
			$zforcehttps = '0';
		}
	} catch (Exception $e) {
		
	}

	$zresponse = array();

	switch ($zfunction) {
		case "downloadqueue":
			$zresponse = $wtw_3dinternet_downloads->addDownloadQueue($zwebid, $zwebtype);
			break;
		case "syncwebsites":
			$zapikeyid = '';
			$zresults = $wtwconnect->query("
				select * 
				from ".wtw_tableprefix."apikeys
				where appurl='".$zstoreurl."'
					and wtwkey='".$zwtwkey."'
					and wtwsecret='".$zwtwsecret."'
					and deleted=0
					and approved=1;");
			foreach ($zresults as $zrow) {
				$zapikeyid = $zrow->apikeyid;
			}
			if ($wtwconnect->hasValue($zapikeyid)) {
				$i = 0;
				/* key-secret combo has access */
				
			}
			break;
		case "createcommunityandbuilding":
			$zresponse = array(
				'serror'=>'',
				'buildingid'=>'',
				'communityid'=>''
			);
			if ($wtwconnect->hasValue($zwtwusertoken)) {
				if ($zhosturl == 'https://3d.walktheweb.com/') {
					/* WalkTheWeb allows all valid users to create a Hosting account */
					$zresults = $wtwconnect->query("
						select u1.*
						from ".wtw_tableprefix."users u1
						where u1.usertoken='".base64_encode($zwtwusertoken)."'
							and u1.deleted=0;");
					foreach ($zresults as $zrow) {
						$zuserid = $zrow["userid"];
					}
					if (isset($zuserid) && !empty($zuserid)) {
						$wtwconnect->addUserRole($zuserid, "Host");
					}
				}

				/* check is the user with the access token has admin or host access */
				$zresults = $wtwconnect->query("
					select u1.*,
						r1.rolename
					from ".wtw_tableprefix."users u1
						inner join ".wtw_tableprefix."usersinroles ur1
							on u1.userid=ur1.userid
						inner join ".wtw_tableprefix."roles r1
							on ur1.roleid=r1.roleid
					where u1.usertoken='".base64_encode($zwtwusertoken)."'
						and u1.deleted=0
						and (r1.rolename like 'admin'
							or r1.rolename like 'host')
						and ur1.deleted=0
						and r1.deleted=0
					order by r1.rolename
					limit 1;");
				foreach ($zresults as $zrow) {
					$zauthenticationok = true;
					$zuserid = $zrow["userid"];
					$zpastuserid = $zrow["pastuserid"];
					/* user found by access token, update the pastuserid (wtwuserid) to the user account as a reference */
					if ($wtwconnect->hasValue($zuserid) && (!isset($zpastuserid) || empty($zpastuserid))) {
						$zresults = $wtwconnect->query("
							update ".wtw_tableprefix."users
							set pastuserid='".$zwtwuserid."',
								updatedate=now(),
								updateuserid='".$zuserid."'
							where userid='".$zuserid."';");
					}
				}
				if ($zauthenticationok == false) {
					$serror = 'User does not have permission on WalkTheWeb Server';
				}
			} else if ($wtwconnect->hasValue($zusertoken)) {
				/* check is the user with the access token has admin or host access */
				$zresults = $wtwconnect->query("
					select u1.*,
						r1.rolename
					from ".wtw_tableprefix."users u1
						inner join ".wtw_tableprefix."usersinroles ur1
							on u1.userid=ur1.userid
						inner join ".wtw_tableprefix."roles r1
							on ur1.roleid=r1.roleid
					where CONVERT(from_base64(u1.wordpresstoken) USING utf8)='".$zusertoken."'
						and u1.deleted=0
						and (r1.rolename like 'admin'
							or r1.rolename like 'host')
						and ur1.deleted=0
						and r1.deleted=0
					order by r1.rolename
					limit 1;");
				foreach ($zresults as $zrow) {
					$zauthenticationok = true;
					$zuserid = $zrow["userid"];
					$zwtwusertoken = $zusertoken;
				}
				if ($zauthenticationok == false) {
					$serror = 'User does not have permission on WalkTheWeb Server';
				}
			}
			if ($wtwconnect->hasValue($zuserid)) {
				$zauthenticationok = true;
			}

			if ($zauthenticationok && isset($zwebname) && !empty($zwebname)) {
				/* reserved words can not be any part of the webname - you can add your own reserved words */
				$zreserved = array('wtw','walktheweb','http3d','https3d');

				/* check if web alias is using a reserved word (not allowed) */
				$zwebtest = str_replace("_","",str_replace(".","",str_replace("-","",$zwebname)));

				$zfound = '0';
				foreach ($zreserved as $zword) {
					if (strpos($zwebtest, $zword) !== false) {
						 $zfound = '1';
					}
				}
				if ($zfound == '0') {
					/* check if web alias is already in use */
					$zresults = $wtwconnect->query("
						select w1.*
						from ".wtw_tableprefix."webaliases w1
						where w1.deleted=0
							and (w1.communitypublishname='".$zwebname."'
								or w1.buildingpublishname='".$zwebname."');");
					if (count($zresults) == 0) {
						/* not found - is available */
						$zwebnameok = true;
					}
				}
				if ($zwebnameok == false) {
					$serror = 'Web Name is already in use.';
				}
			}

			if ($zauthenticationok && $zwebnameok) {
				if ($wtwconnect->isUserInRole("Host")) {
					$zhostuserid = $zuserid;
				}

				/* download community */
				$zresults = $wtw_3dinternet_downloads->downloadWeb($zcommunityid, $zcommunityid, 'community', $zwtwusertoken, '', '', '', 0, 0, 0, 1, 1, 1, 0, 0, 0);
				
				$znewcommunityid = $zresults["newwebid"];

				if (!isset($znewcommunityid) || empty($znewcommunityid)) {
					$serror = '3D Community Scene could not be created.';
				} else {
					$wtwconnect->query("
						update ".wtw_tableprefix."communities
							set communityname='".addslashes($zwtwstorename)." 3D Community'
							where communityid='".$znewcommunityid."'
							limit 1;
					");
				}
				
				/* get building start position and rotation */
				$zbuildingpositionx = 0;
				$zbuildingpositiony = 0;
				$zbuildingpositionz = 0;
				$zbuildingscalingx = 1;
				$zbuildingscalingy = 1;
				$zbuildingscalingz = 1;
				$zbuildingrotationx = 0;
				$zbuildingrotationy = 0;
				$zbuildingrotationz = 0;
				$zresults = $wtwconnect->query("
					select c1.*
					from ".wtw_tableprefix."communities c1
					where c1.deleted=0
						and c1.communityid='".$znewcommunityid."';");
				foreach ($zresults as $zrow) {
					$zbuildingpositionx = $zrow["buildingpositionx"];
					$zbuildingpositiony = $zrow["buildingpositiony"];
					$zbuildingpositionz = $zrow["buildingpositionz"];
					$zbuildingscalingx = $zrow["buildingscalingx"];
					$zbuildingscalingy = $zrow["buildingscalingy"];
					$zbuildingscalingz = $zrow["buildingscalingz"];
					$zbuildingrotationx = $zrow["buildingrotationx"];
					$zbuildingrotationy = $zrow["buildingrotationy"];
					$zbuildingrotationz = $zrow["buildingrotationz"];
				}

				/* download building */
				$zresults = $wtw_3dinternet_downloads->downloadWeb($zbuildingid, $zbuildingid, 'building', $zwtwusertoken, $znewcommunityid, 'community', $znewcommunityid, $zbuildingpositionx, $zbuildingpositiony, $zbuildingpositionz, $zbuildingscalingx, $zbuildingscalingy, $zbuildingscalingz, $zbuildingrotationx, $zbuildingrotationy, $zbuildingrotationz);
				
				$znewbuildingid = $zresults["newwebid"];
				
				if (!isset($znewbuildingid) || empty($znewbuildingid)) {
					$serror = '3D Shopping Building could not be created.';
				} else {
					$wtwconnect->query("
						update ".wtw_tableprefix."buildings
							set buildingname='".addslashes($zwtwstorename)."'
							where buildingid='".$znewbuildingid."'
							limit 1;
					");
				}
				/* add webalias for new community to map the web url to the new 3D Website */
				$zwebaliasid = $wtwconnect->getRandomString(16,1);
				$wtwconnect->query("
					insert into ".wtw_tableprefix."webaliases
					   (webaliasid,
					    hostuserid,
						forcehttps,
						domainname,
						webalias,
						communityid,
						communitypublishname,
						buildingid,
						buildingpublishname,
						thingid,
						thingpublishname,
						createdate,
						createuserid,
						updatedate,
						updateuserid)
					values
					   ('".$zwebaliasid."',
					    '".$zhostuserid."',
						".$zforcehttps.",
						'".$zdomainname."',
						'".$zdomainname."',
						'".$znewcommunityid."',
						'".$zwebname."',
						'',
						'',
						'',
						'',
						now(),
						'".$zuserid."',
						now(),
						'".$zuserid."');");

				/* add webalias for new 3D Building so it can be opened directly */ 
				$zwebaliasid = $wtwconnect->getRandomString(16,1);
				$wtwconnect->query("
					insert into ".wtw_tableprefix."webaliases
					   (webaliasid,
					    hostuserid,
						forcehttps,
						domainname,
						webalias,
						communityid,
						communitypublishname,
						buildingid,
						buildingpublishname,
						thingid,
						thingpublishname,
						createdate,
						createuserid,
						updatedate,
						updateuserid)
					values
					   ('".$zwebaliasid."',
					    '".$zhostuserid."',
						".$zforcehttps.",
						'".$zdomainname."',
						'".$zdomainname."',
						'',
						'',
						'".$znewbuildingid."',
						'".$zwebname."',
						'',
						'',
						now(),
						'".$zuserid."',
						now(),
						'".$zuserid."');");
				
				/* if store key and secret exist - check if plugin is installed */
				if ($wtwconnect->hasValue($zstoreurl) && $wtwconnect->hasValue($zwookey) && $wtwconnect->hasValue($zwoosecret)) {

					/* check if store tables exist (3D Shopping Plugin exists) */
					$zstoretables = 0;
					$zresults = $wtwconnect->query("
						select count(*) as scount
						from information_schema.tables 
						where table_schema = '".wtw_dbname."'
							and (table_name = '".wtw_tableprefix."shopping_stores'
								or table_name = '".wtw_tableprefix."shopping_connectstores');");
					foreach ($zresults as $zrow) {
						$zstoretables = $zrow["scount"];
					}
					/* if store tables exist - add permission entries */
					if ($zstoretables > 1) {
						$zstoreid = $wtwconnect->getRandomString(16,1);
						$wtwconnect->query("
							insert into ".wtw_tableprefix."shopping_stores
								(storeid,
								 storename,
								 storeiframes,
								 storeurl,
								 storecarturl,
								 storeproducturl,
								 woocommerceapiurl,
								 woocommercekey, 
								 woocommercesecret,
								 approveddate,
								 approveduserid,
								 createdate,
								 createuserid,
								 updatedate,
								 updateuserid)
							values
								('".$zstoreid."',
								 '".base64_encode($zwtwstorename)."',
								 ".$ziframes.",
								 '".$zstoreurl."',
								 '".$zstorecarturl."',
								 '".$zstoreproducturl."',
								 '".$zstoreapiurl."',
								 '".base64_encode($zwookey)."', 
								 '".base64_encode($zwoosecret)."',
								 now(),
								 '".$zuserid."',
								 now(),
								 '".$zuserid."',
								 now(),
								 '".$zuserid."');");
						/* connect store settings to 3D Store Building */
						$zconnectid = $wtwconnect->getRandomString(16,1);
						$wtwconnect->query("
							insert into ".wtw_tableprefix."shopping_connectstores
								(connectid,
								 storeid,
								 communityid,
								 buildingid,
								 thingid,
								 createdate,
								 createuserid,
								 updatedate,
								 updateuserid)
								values
								('".$zconnectid."',
								 '".$zstoreid."',
								 '',
								 '".$znewbuildingid."',
								 '',
								 now(),
								 '".$zuserid."',
								 now(),
								 '".$zuserid."');");
					} else {
						$serror = '3D Shopping Plugin is not installed.';
					}
				}
				
				/* set the return values */
				$zresponse = array(
					'serror'=>$serror,
					'buildingid'=>$znewcommunityid,
					'communityid'=>$znewbuildingid
				);
			} else {
				$zresponse = array(
					'serror'=>$serror,
					'buildingid'=>'',
					'communityid'=>''
				);
			}
			break;
	}
	echo json_encode($zresponse);	
} catch (Exception $e) {
	$wtwconnect->serror("connect-wordpress.php=".$e->getMessage());
}
?>
----------------------
