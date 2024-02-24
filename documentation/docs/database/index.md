# DB

## Original

Here is a first attempt design of the structure for tracking irs or itns campaigns. before each table or key field a comment describing key details about the table/field, I need you first to understand those comment in this NMCP campaigns context and revise them, you can change where to put them, write them better, make them clear and to the point. and respond with the same design with the revised comments

```sql
-- `m_campaign` campaigns information, database will be used for multiple campaigns (i.e itns campaign_type_id =1, irs campaign_type_id=2)
CREATE TABLE 
    m_campaign 
    ( 
        campaign_code         CHARACTER VARYING(64) NOT NULL, 
        campaign_started_date INTEGER NOT NULL, 
        campaign_days         INTEGER, 
        campaign_type_id      INTEGER, 
        campaign_year         INTEGER, 
        campaign_id           BIGINT DEFAULT nextval('m_campaign_campaign_serial_seq'::regclass) 
        NOT NULL, 
        campaign_note CHARACTER VARYING(255), 
        CONSTRAINT campaign_started_date_fk FOREIGN KEY (campaign_started_date) REFERENCES 
        "m_date_dimension" ("date_id"), 
        UNIQUE (campaign_code), 
        UNIQUE (campaign_id) 
    );
    -- `m_villages_locations` All country locations from which certain locations in a certain campaign, during planning are chosen to be targeted and are imported into the `daily_targets`. this table does not directly have inside it foreign keys to any campaign related table. but rather the other tables reference it, such as the `daily_targets`, the `c_kobo_irs_team_main` and `c_kobo_itns_team_main` campaigns data tables, and others.
    CREATE TABLE 
    m_villages_locations 
    ( 
        location_id             BIGINT DEFAULT nextval('location_id_serial_seq'::regclass) NOT NULL, 
        location_district_name CHARACTER VARYING(255), 
        location_ppc_code         BIGINT NOT NULL, 
        location_subdistrict_name CHARACTER VARYING(255), 
        location_village_name     CHARACTER VARYING(255), 
        location_subvillage_name  CHARACTER VARYING(255), 
        location_longitude DOUBLE PRECISION, 
        location_latitude DOUBLE PRECISION, 
        CONSTRAINT villages_locations_pkey PRIMARY KEY (location_id), 
        REFERENCES "m_districts" ("district_id"), 
        CONSTRAINT village_id_ux UNIQUE (location_id), 
        CONSTRAINT village_ppc_code_ux UNIQUE (location_ppc_code) 
    );

    -- `c_region` Only During IRS, in the planning phase the locations in the daily targets are divided into regions, each region encompasses multiple location and have one or more sub-regions represented by a warehouse/distribution_point each with a supervisor, the supervisor of each warehouse/distribution_point supervise the spraying teams responsible for the covering of the locations belonging to this warehouse. region-warehouse/distribution_point-teams-locations-day relations are multiply defined in the `c_daily_targets` as well as the `c_warehouses_distribution_points` and `c_team`, I don't know if this is a good idea or it's better to only have one place representing this relation.
    CREATE TABLE 
    c_region 
    ( 
        region_id          BIGINT DEFAULT nextval('c_region_serial_seq'::regclass) NOT NULL, 
        region_name        CHARACTER VARYING(255), 
        region_head        CHARACTER VARYING(255), 
        region_notes       CHARACTER VARYING(255), 
        region_campaign_id BIGINT NOT NULL, 
        region_name_en     CHARACTER VARYING(255), 
        region_number      INTEGER NOT NULL, 
        PRIMARY KEY (region_id), 
        CONSTRAINT c_region_campaign_id_fk FOREIGN KEY (region_campaign_id) REFERENCES 
        "m_campaign" ("campaign_id") 
    );

-- `c_warehouses_distribution_points` Distribution Points: These are specific locations within a community where the nets(itn campaigns)/pesticides(IRS campaigns) will be handed out to teams. These points can be schools, community centers, health clinics which are selected and the stock is moved before the start of the campaign during the planning phase. `c_warehouses_distribution_points` list all campaigns warehouses/distribution_points, for ITNs campaigns and IRS campaigns warehouses, taking into account that in ITNs campaigns there are no regions divisions so `wh_region_id` is null
CREATE TABLE 
    c_warehouses_distribution_points 
    ( 
        wh_id             BIGINT DEFAULT nextval('c_warehouses_serial_seq'::regclass) NOT NULL, 
        wh_name           CHARACTER VARYING(255), 
        wh_description    CHARACTER VARYING(255), 
        wh_gps_coordinate CHARACTER VARYING(255) DEFAULT nextval('c_warehouses_serial_seq':: 
        regclass), 
        wh_campaign_id     BIGINT, 
        wh_region_id       BIGINT, 
        CONSTRAINT c_warehouses_pkey PRIMARY KEY (wh_id), 
        CONSTRAINT wh_campaign_id_fk FOREIGN KEY (wh_campaign_id) REFERENCES "m_campaign" 
        ("campaign_id"), 
        CONSTRAINT wh_region_id_fk FOREIGN KEY (wh_region_id) REFERENCES "c_region" ("region_id"), 
    );

-- `c_team` a team represent either a leader of a group of workers (called team/Foreman) responsible for spraying (in IRS campaigns) or distributing itns (in ITNs campaigns) and he sends the spraying data that are stored in the `c_kobo_irs_team_main`, or a wh_distribution_point's supervisor supervising a group of teams/Foremen in a particular warehouse/distribution_point, a supervisor also sends the same data his teams/foremen send and stored in the same table of foremen data `c_kobo_irs_team_main` as (duplicate for comparisons with data sent by each one of his foremen for checking). `c_team` list any campaigns teams, either ITNs campaigns or IRS campaigns teams but in ITNs campaigns locations are not divided into regions, the warehouse/distribution_point is considered the division-point (summary reporting point) of the area encompassing the locations it covers, so in ITNs campaigns `wh_region_id` is null, team_type is used to differentiate foreman from supervisor containing either [FOREMAN, or SUPERVISOR]
CREATE TABLE 
    c_team 
    ( 
        team_id             BIGINT DEFAULT nextval('c_team_new_serial_seq'::regclass) NOT NULL, 
        team_contact_person TEXT, 
        team_wh_id          BIGINT, 
        team_number         BIGINT,
        team_type team_type,
        team_region_id           BIGINT, 
        CONSTRAINT c_team_new_pkey PRIMARY KEY (team_id), 
        CONSTRAINT c_team_region_id_fk FOREIGN KEY (team_region_id) REFERENCES "c_region" ("region_id"), 
        CONSTRAINT c_team_wh_id_fk FOREIGN KEY (team_wh_id) REFERENCES "c_warehouses" ("wh_id"), 
    );

-- `c_daily_targets` Daily Operational Plan: Assigned Areas: Spray teams are assigned specific locations/villages to visit their households to cover each day. These assignments are designed to ensure systematic coverage of the targeted communities.
CREATE TABLE 
    c_daily_targets 
    ( 
        -- within which distribution_points
        target_warehouse_id BIGINT NOT NULL, 
        -- assigned team, the location sometimes might be covered and its data is sent by a different team
        target_team_id      BIGINT NOT NULL, 
        -- estimated population of the location
        target_population DOUBLE PRECISION, 
        -- campaign id
        target_campaign_id BIGINT NOT NULL, 
        -- day at which the team is plan to visit this location, the team might visit in a different day 
        target_day_date_id INTEGER NOT NULL, 
        target_location_id BIGINT NOT NULL, 
        -- estimated number of houses
        target_houses DOUBLE PRECISION, 
        -- estimated room average area of the houses in this location
        target_room_avg_area DECIMAL NOT NULL,
        -- estimated number of rooms in this location
        target_rooms INTEGER NOT NULL,
        -- estimated pesticides needed to cover the total area in this location,it's calculated using: ((target_rooms * room_avg_area) / 250
        target_estimated_pesticides_sachets INTEGER NOT NULL,

        target_region_id   BIGINT,

        CONSTRAINT c_villagetargetinfo_target_campaign_id_fkey FOREIGN KEY 
        (target_campaign_id) REFERENCES "m_campaign" ("campaign_id"), 
 
        CONSTRAINT c_villagetargetinfo_target_team_id_fkey FOREIGN KEY (target_team_id) 
        REFERENCES "c_team" ("team_id"), 
        CONSTRAINT c_villagetargetinfo_target_warehouse_id_fkey FOREIGN KEY (target_warehouse_id) REFERENCES "c_warehouses_distribution_points" ("wh_id"), 
        CONSTRAINT c_villagetargetinfo_day_date_id_fkey FOREIGN KEY (target_day_date_id) REFERENCES "m_date_dimension" ("date_id"),    
        CONSTRAINT c_villagetargetinfo_location_id_fkey FOREIGN KEY (target_location_id) REFERENCES "m_villages_locations" ("location_id") 
    );

-- Bellow tables are related to Data Collection and Reporting: `c_kobo_irs_team_main` record detailed information on the households sprayed, quantities of insecticide used, areas covered, and any refusals or missed households per day and house and, it contains a merged data sent for the same location of the same data one sent by team/Foreman type and one sent by a team/foreman's supervisor for comparison, and differentiated by the `report_team_id`, we can choose to report summaries using either one type of team. a location might has multiple reports if covered more than one day, the progress status that is considered when needed is the last report progress_status based on _submission_time. 
CREATE TABLE 
    c_kobo_irs_team_main 
    ( 
        _id                                     BIGINT NOT NULL, 
        _uuid                                   CHARACTER VARYING(255) NOT NULL, 
        _submission_time                        TIMESTAMP(6) WITHOUT TIME ZONE, 

        -- one of multiple status of the statuses [ full, ongoing, partial, non_sprayed_for_reason]
        progress_status                         CHARACTER VARYING(255) NOT NULL, 
        -- if progress_status is non_sprayed_for_reason, here is the reason which is on of [displaced_location, not_sprayed_for_a_reason, rejection, war_zone_location, other_reason], houses and rooms data will all be zero, we can use the values from the daily_targets if we want to calculate something
        report_unspray_reason                   CHARACTER VARYING(2000), 

        location_id                             BIGINT NOT NULL, 

        `report_campaign_id` is represented in multiple tables, it makes some queries easier and faster, there might be a better way to not repeat this.
        report_campaign_id                      BIGINT NOT NULL, 

        -- the team covered this location and sent this report
        report_team_id                          BIGINT NOT NULL, 

        -- the day at which the spray of the houses of this location took place
        day_date_id                             INTEGER NOT NULL, 

        -- total houses visited
        report_houses_total_entered             INTEGER, 

        -- houses sprayed of total visited
        report_houses_sprayed_entered           INTEGER,

        -- houses non-sprayed of total visited
        report_houses_nonsprayed_entered        INTEGER, 

        -- houses refused of non-sprayed
        report_houses_refused_entered           INTEGER, 

        report_houses_closed_entered            INTEGER, 
        report_population_in_sprayed_entered    INTEGER, 
        -- total rooms in the houses visited
        report_rooms_total_entered              INTEGER, 
        -- rooms sprayed of total rooms
        report_rooms_sprayed_entered            INTEGER, 
        -- houses non-sprayed of total rooms
        report_rooms_nonsprayed_entered         INTEGER, 
        -- total consumed pesticides in sachets, each sachet covers area of 250 m2 of walls
        report_total_sachets_entered            INTEGER, 
        -- gps location of the village, for tracking and checking
        report_captured_gps                     CHARACTER VARYING(255), 
        _database_created_at  TIMESTAMP(6) WITHOUT TIME ZONE DEFAULT CURRENT_TIMESTAMP, 
        CONSTRAINT c_kobo_irs_team_main_2_pkey PRIMARY KEY (_id), 
        CONSTRAINT c_irs_kobo2_campaign_id_fk FOREIGN KEY (report_campaign_id) REFERENCES 
        "m_campaign" ("campaign_id"), 
        CONSTRAINT c_irs_kobo2_day_date_id_fk FOREIGN KEY (day_date_id) REFERENCES 
        "m_date_dimension" ("date_id"), 
        CONSTRAINT c_irs_kobo2_location_id_fk FOREIGN KEY (location_id) REFERENCES 
        "m_villages_locations" ("location_id"), 
        CONSTRAINT c_irs_kobo2_team_id_fk FOREIGN KEY (report_team_id) REFERENCES "c_team" 
        ("team_id"), 
        CONSTRAINT c_irs_kobo_kobo2_uuid_u UNIQUE (_uuid) 
    );

    -- Logistics and Supply Chain Management, this records the daily transactions of a warehouse/distribution-point. `wh_transaction_type` on of [team, warehouse] a team if the movement from the distribution point to a team, and a warehouse if the transaction is from a warehouse/distribution-point to another warehouse/distribution-point
    CREATE TABLE 
    c_wh_transactions 
    ( 
        transaction_id BIGINT DEFAULT nextval('transactions_transaction_id_seq'::regclass) NOT 
        NULL, 
        campaign_type    BIGINT NOT NULL, 
        item_id          BIGINT NOT NULL, 
        source_id        BIGINT, 
        destination_id   BIGINT, 
        source_type      BIGINT, 
        destination_type BIGINT, 
        quantity         INTEGER NOT NULL, 
        transaction_type wh_transaction_type NOT NULL, 
        note             CHARACTER VARYING(255), 
        reference_id     BIGINT, 
        transaction_date TIMESTAMP(6) WITHOUT TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL, 
        CONSTRAINT transactions_pkey PRIMARY KEY (transaction_id), 
        CONSTRAINT transactions_campaign_id_fkey FOREIGN KEY (campaign_type) REFERENCES "m_campaign" ("campaign_id"), 
        CONSTRAINT transactions_destination_type_id_fkey FOREIGN KEY (destination_type) REFERENCES "c_wh_ledger_entity_type" ("type_id"), 
        CONSTRAINT transactions_item_id_fkey FOREIGN KEY (item_id) REFERENCES "c_inventory_item" ("item_id"), 
        CONSTRAINT transactions_source_type_id_fkey FOREIGN KEY (source_type) REFERENCES "c_wh_ledger_entity_type" ("type_id") 
    );
-- other tables such as warehouse balance, movements, c_kobo_itns_team_main storing foremen submitted data when the campaign is ITNs... etc

```

## Revised documenting

The provided SQL schema outlines a database structure designed to support the management and tracking of National Malaria Control Program (NMCP) campaigns, specifically focusing on Indoor Residual Spraying (IRS) and Insecticide-Treated Nets (ITNs) distributions. Here's a detailed description of the database schema based on the revised comments:

### Overview

The schema is organized to facilitate the planning, execution, and reporting of malaria control campaigns. It encompasses various aspects of these campaigns, including:

- **Campaign Details:** Information about each campaign, such as type (IRS or ITNs), start date, duration, and additional notes.
- **Geographic Targeting:** Records of potential target locations (villages) across the country, including geographic coordinates and administrative divisions.
- **Operational Planning:** Division of targeted areas into regions for logistical purposes in IRS campaigns, assignment of distribution points (warehouses), and daily operational plans specifying which teams visit which locations.
- **Team Management:** Information about teams responsible for spraying or net distribution, including their leaders and contact details.
- **Data Collection and Reporting:** Detailed records of daily activities by teams, including household visits, insecticide usage, and reasons for any houses not sprayed.
- **Logistics and Supply Chain Management:** Tracking of inventory movements, including supplies distributed to teams or transferred between warehouses.

### Key Components

1. **Campaign Information (`m_campaign`):** This table is central to the schema, categorizing each campaign by type (e.g., IRS or ITNs), year, and other identifying details. It enables the tracking of multiple, simultaneous campaigns.

2. **Locations (`m_villages_locations`):** A comprehensive list of all potential target locations within the country, which can be selected during campaign planning. This table serves as a reference for planning and reporting, without being directly tied to a specific campaign.

3. **Regions (`c_region`):** Specifically for IRS campaigns, regions are defined to streamline planning and resource allocation. Each region can encompass multiple locations and is associated with a specific campaign.

4. **Warehouses/Distribution Points (`c_warehouses_distribution_points`):** Identified locations where campaign materials (pesticides for IRS or nets for ITNs) are stored and from where they are distributed to operational teams. These points are crucial for logistical planning.

5. **Teams (`c_team`):** Details about the teams conducting the spraying or net distribution, including the differentiation between foremen (team leaders) and supervisors for reporting and operational clarity.

6. **Targets (`c_daily_targets`):** Operational plans detailing which teams are assigned to which targeted-location on specific days, including estimates of population, housing, and the resources required to achieve campaign goals.

7. **Daily Targets (`c_daily_targets`):** Operational plans detailing which teams are assigned to which targeted-location on specific days, including estimates of population, housing, and the resources required to achieve campaign goals.

8. **Data Collection and Reporting (`c_kobo_irs_team_main`):** Captures detailed reports from the field, including outcomes of daily operations, reasons for any deviations from the plan, and GPS data for verification purposes.

9. **Logistics and Transactions (`c_wh_transactions`):** Records all movements of campaign materials, whether to teams for daily operations or between warehouses, ensuring accurate tracking of inventory and supplies.

### Purpose and Utility

This schema is designed to support comprehensive tracking and management of NMCP campaigns, facilitating data-driven decision-making and operational efficiency. By structuring data around campaigns, locations, teams, and logistics, the schema provides a solid foundation for monitoring progress, evaluating impact, and planning future campaigns. It allows for detailed reporting and analysis, which are crucial for adjusting strategies, ensuring resource availability, and achieving the overall goal of malaria control and eradication.

### The Database tables SQL creation

```sql

-- `m_campaigntypes`: (e.g., ITNs, IRS)

CREATE TABLE 
    m_campaigntypes 
    ( 
        campaign_type_id INTEGER DEFAULT nextval('m_campaign_type_serial_seq'::regclass) NOT NULL, 
        Campaign_type    CHARACTER VARYING(255) NOT NULL, 
        description      CHARACTER VARYING(255), 
        CONSTRAINT campaign_type_id_ux PRIMARY KEY (campaign_type_id), 
        CONSTRAINT campaign_type_ux UNIQUE ("Campaign_type") 
    );

-- `m_campaign`: Stores information about each campaign. It supports multiple types of campaigns (e.g., ITNs with `campaign_type_id`=1, IRS with `campaign_type_id`=2), allowing the database to track various intervention strategies.

CREATE TABLE 
    m_campaign 
    ( 
        campaign_code         CHARACTER VARYING(64) NOT NULL, 
        campaign_started_date TIMESTAMP(6) WITHOUT TIME ZONE, 
        campaign_days         INTEGER, 
        campaign_type_id      INTEGER NOT NULL, 
        campaign_id           BIGINT DEFAULT nextval('m_campaign_campaign_serial_seq'::regclass) 
        NOT NULL, 
        campaign_note CHARACTER VARYING(255), 

        PRIMARY KEY (campaign_id), 
        CONSTRAINT campaign_type_fk FOREIGN KEY (campaign_type_id) 
        REFERENCES "m_campaigntypes" ("campaign_type_id"), 
        UNIQUE (campaign_code)
    );

-- `m_villages_locations`: Catalogs potential target locations across the country. Selections for specific campaigns are made during planning and are utilized in tables like `daily_targets` and for reporting in `c_kobo_irs_team_main` and `c_kobo_itns_team_main`.

CREATE TABLE m_villages_locations
(
    location_id                BIGINT DEFAULT nextval('location_id_serial_seq'::regclass) NOT NULL,
    location_district_id     BIGINT NOT NULL,
    location_ppc_code          BIGINT NOT NULL,
    location_subdistrict_name  CHARACTER VARYING(255),
    location_village_name      CHARACTER VARYING(255),
    location_subvillage_name   CHARACTER VARYING(255),
    location_longitude         DOUBLE PRECISION,
    location_latitude          DOUBLE PRECISION,
    CONSTRAINT villages_locations_pkey PRIMARY KEY (location_id),
    CONSTRAINT villages_locations_district_id_fk FOREIGN KEY (location_district_id) 
    REFERENCES "m_districts" ("district_id"),
    CONSTRAINT villages_locations_ppc_code_ux UNIQUE (location_ppc_code)
);

-- `c_region`: Defines regions for IRS campaigns, where each encompasses multiple locations. These regions are used for logistical planning, including warehouse/distribution point assignments and team management. The structure supports nested regions through warehouse and team assignments. For IRS campaigns only, regions are defined in this table, each linked to a specific campaign.

CREATE TABLE c_region
(
    region_id          BIGINT DEFAULT nextval('c_region_serial_seq'::regclass) NOT NULL,
    region_name        CHARACTER VARYING(255),
    region_head        CHARACTER VARYING(255),
    region_notes       CHARACTER VARYING(255),
    region_campaign_id BIGINT NOT NULL,
    region_name_en     CHARACTER VARYING(255),
    region_number      INTEGER NOT NULL,
    PRIMARY KEY (region_id),
    CONSTRAINT c_region_campaign_id_fk FOREIGN KEY (region_campaign_id) 
    REFERENCES "m_campaign" ("campaign_id")
);

-- `c_warehouses_distribution_points`: Lists distribution points for both ITNs and IRS campaigns, indicating where resources are staged for distribution to teams. ITNs campaigns do not use regional divisions, hence `wh_region_id` can be null for them.

CREATE TABLE c_warehouses_distribution_points
(
    wh_id              BIGINT DEFAULT nextval('c_warehouses_serial_seq'::regclass) NOT NULL,
    wh_name            CHARACTER VARYING(255),
    wh_description     CHARACTER VARYING(255),
    wh_gps_coordinate  CHARACTER VARYING(255),
    wh_campaign_id     BIGINT,
    wh_region_id       BIGINT,
    CONSTRAINT c_warehouses_pkey PRIMARY KEY (wh_id),
    CONSTRAINT wh_campaign_id_fk FOREIGN KEY (wh_campaign_id) 
    REFERENCES "m_campaign" ("campaign_id"),
    CONSTRAINT wh_region_id_fk FOREIGN KEY (wh_region_id) 
    REFERENCES "c_region" ("region_id")
);

-- `c_team`: Represents teams tasked with either spraying (IRS) or distributing nets (ITNs), including team leaders and supervisors. The crucial team_type field differentiates between foreman teams directly involved in spraying and supervisor teams overseeing multiple foreman teams and consolidating their reports. it has either [FOREMAN, SUPERVISOR].

CREATE TABLE c_team
(
    team_id            BIGINT DEFAULT nextval('c_team_new_serial_seq'::regclass) NOT NULL,
    team_contact_person TEXT,
    team_wh_id         BIGINT NOT NULL,
    team_number        BIGINT NOT NULL,
    team_type          team_type, 
    CONSTRAINT c_team_new_pkey PRIMARY KEY (team_id),
    CONSTRAINT c_team_wh_id_fk FOREIGN KEY (team_wh_id) 
    REFERENCES "c_warehouses_distribution_points" ("wh_id")
);

-- `c_targets_master`: moving target_warehouse_id, target_location_id, target_region_id from c_daily_targets to this table, this would contain the target resulted from planning phase, after choosing locations and deciding on the regions and warehouses/distribution-point 

CREATE TABLE 
    c_targets_master 
    ( 
        target_id          BIGINT DEFAULT nextval('c_targets_master_serial_seq'::regclass) NOT NULL, 
        target_warehouse_id BIGINT NOT NULL,
        target_location_id  BIGINT NOT NULL, 
        target_region_id    BIGINT,
        target_field_code   BIGINT NOT NULL, 
        location_longitude DOUBLE PRECISION, 
        location_latitude DOUBLE PRECISION, 
        PRIMARY KEY (target_id), 
        CONSTRAINT c_targets_master_target_location_id_fkey FOREIGN KEY (target_location_id) 
        REFERENCES "m_villages_locations" ("location_id"), 
        CONSTRAINT c_targets_master_target_region_id_fkey FOREIGN KEY (target_region_id) 
        REFERENCES "c_region" ("region_id"), 
        CONSTRAINT c_targets_master_target_warehouse_id_fkey FOREIGN KEY (target_warehouse_id) 
        REFERENCES "c_warehouses_distribution_points" ("wh_id"), 
        UNIQUE (target_region_id, target_warehouse_id, target_location_id)
        UNIQUE (target_region_id, target_warehouse_id, target_field_code)
    );
    
-- `c_daily_targets`: 
-- This table defines the operational plan for each day within an IRS campaign.
-- It links targets from `c_targets_master` to specific teams (`c_team`) on specific dates (`m_date_dimension`).
-- Estimated resources and coverage are planned here, but actual execution details are captured in `c_kobo_irs_team_main`.

-- Key points:
-- * Estimated values based on planning, actual execution details in `c_kobo_irs_team_main`.
-- * Divergences from planned teams/dates captured in `c_kobo_irs_team_main`.
-- * Multiple reports per target on different days possible in `c_kobo_irs_team_main`.
-- * Progress tracking: Latest report's progress_status is considered for progress tracking.

-- Fields:

-- * target_team_id: The team assigned to the target for this day. (FK to c_team)
-- * target_day_date_id: The date ID for the planned spraying day. (FK to m_date_dimension)
-- * target_population (optional): Estimated population of the target location.
-- * target_houses: Estimated number of houses in the target location.
-- * target_rooms: Estimated number of rooms in the target location.
-- * target_room_avg_area: Average area of a room in the target location.
--     Used to estimate total area requiring insecticide coverage.
-- * target_estimated_pesticides_sachets: Estimated number of insecticide sachets
--     needed for the target location, based on `target_room_avg_area` and
--     `target_rooms`, assuming a specific coverage area per sachet.
-- * no_of_team_spray_workers: Number of spray workers assigned to the target.
-- * area_covered_per_worker_per_day: Estimated area covered by one worker
--     in a day. Used to assess workload and monitor progress.

CREATE TABLE c_daily_targets
(
    target_team_id                BIGINT NOT NULL,
    target_population             DOUBLE PRECISION,
    target_day_date_id            INTEGER NOT NULL,
    target_id            BIGINT NOT NULL,
    target_houses                 DOUBLE PRECISION,
    target_rooms                  INTEGER NOT NULL,
    target_room_avg_area          DECIMAL NOT NULL,
    target_estimated_pesticides_sachets INTEGER NOT NULL,
    no_of_team_spray_workers                  INTEGER NOT NULL, 
    area_covered_per_worker_per_day                 INTEGER NOT NULL,
    CONSTRAINT c_targets_master_id_fkey FOREIGN KEY (target_id) 
    REFERENCES "c_targets_master" ("target_id"),
    CONSTRAINT c_villagetargetinfo_copy1_target_team_id_fkey FOREIGN KEY (target_team_id) 
    REFERENCES "c_team" ("team_id"),
    CONSTRAINT c_villagetargetinfo_day_date_id_fkey FOREIGN KEY (target_day_date_id) 
    REFERENCES "m_date_dimension" ("date_id")
);

-- `c_kobo_irs_team_main`:
-- This table captures detailed field reports submitted by spray teams during IRS campaigns.
-- It provides granular data on sprayed households, insecticide usage, and coverage per house.

-- Fields:

-- * _id: Unique identifier for the report.
-- * _uuid: Unique identifier for the data submission.
-- * _submission_time: Timestamp when the report was submitted.
-- * progress_status: One of "full", "ongoing", "partial", or "non_sprayed_for_reason".
-- * report_unspray_reason (optional): Reason for non-spraying if
--     progress_status is "non_sprayed_for_reason".
-- * target_id: FK to the target location in `c_targets_master`.
-- * report_team_id: The team that covered the location and submitted the report.
--     (FK to c_team)
-- * day_date_id: The date the spraying occurred.
-- * report_houses_total_entered: Total number of houses visited.
-- * report_houses_sprayed_entered: Number of houses sprayed out of total visited.
-- * report_houses_nonsprayed_entered: Number of houses not sprayed out of total visited.
-- * report_houses_refused_entered: Number of houses that refused spraying.
-- * report_houses_closed_entered: Number of houses that were closed.
-- * report_population_in_sprayed_entered: Estimated population in sprayed houses.
-- * report_rooms_total_entered: Total number of rooms in visited houses.
-- * report_rooms_sprayed_

CREATE TABLE c_kobo_irs_team_main
(
    _id                           BIGINT NOT NULL,
    _uuid                         CHARACTER VARYING(255) NOT NULL,
    _submission_time              TIMESTAMP(6) WITHOUT TIME ZONE, 
    progress_status               CHARACTER VARYING(255) NOT NULL,
    report_unspray_reason         CHARACTER VARYING(2000),
    target_id                   BIGINT NOT NULL,
    report_team_id            BIGINT NOT NULL, 
    day_date_id                   INTEGER NOT NULL,
    report_houses_total_entered   INTEGER,
    report_houses_sprayed_entered INTEGER,
    report_houses_nonsprayed_entered INTEGER, 
    report_houses_refused_entered INTEGER, 
    report_houses_closed_entered  INTEGER,
    report_population_in_sprayed_entered INTEGER,
    report_rooms_total_entered    INTEGER, 
    report_rooms_sprayed_entered  INTEGER, 
    report_rooms_nonsprayed_entered INTEGER, 
    report_total_sachets_entered  INTEGER, 
    report_captured_gps           CHARACTER VARYING(255), 
    _database_created_at          TIMESTAMP(6) WITHOUT TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT c_kobo_irs_team_main_2_pkey PRIMARY KEY (_id),
    CONSTRAINT c_irs_kobo2_day_date_id_fk FOREIGN KEY (day_date_id) REFERENCES "m_date_dimension" ("date_id"),
    CONSTRAINT c_irs_kobo2_target_id_fk FOREIGN KEY (target_id) REFERENCES "c_targets_master" ("target_id"),
    CONSTRAINT c_irs_kobo2_team_id_fk FOREIGN KEY (report_team_id) REFERENCES "c_team" ("team_id"),
    CONSTRAINT c_irs_kobo_kobo2_uuid_u UNIQUE (_uuid)
);

-- `m_villages_de` data elements table, such as Houses, Population, rooms. it was created due to the rigid previous way making dataelements defined in a separately from other tables.

CREATE TABLE 
    m_villages_de 
    ( 
        dataelement_id        BIGINT DEFAULT nextval('m_villages_de_serial_seq'::regclass) NOT NULL, 
        dataelement_name        CHARACTER VARYING(255) NOT NULL, 
        dataelement_description CHARACTER VARYING(255), 
        CONSTRAINT m_villages_dataelemnts_copy1_pkey1 PRIMARY KEY (dataelement_id) 
    );

-- `m_villages_de_value_source` a one data element can have more than one value depending on the source of info, in targets we can choose the quantity is coming from which source.

CREATE TABLE 
    m_villages_de_value_source 
    ( 
        value_source_id BIGINT DEFAULT nextval('m_villages_de_value_source_serial_seq'::regclass) 
        NOT NULL, 
        value_source_name CHARACTER VARYING(255) NOT NULL, 
        CONSTRAINT m_villages_dataelemnts_copy1_pkey PRIMARY KEY (value_source_id), 
        CONSTRAINT m_dataelement_source_name_ux UNIQUE (value_source_name) 
    );

-- `m_villages_de_values` available values of data elements for a location in the `m_villages_locations` master list by which source in the `m_villages_de_value_source`.

CREATE TABLE 
    m_villages_de_values 
    ( 
        value_source_id BIGINT NOT NULL, 
        value_de_id     BIGINT NOT NULL, 
        value_date_id   INTEGER, 
        dataelement_value DOUBLE PRECISION NOT NULL, 
        value_location_id BIGINT NOT NULL, 
        CONSTRAINT m_villages_values_date_id_fk FOREIGN KEY (value_date_id) REFERENCES 
        "m_date_dimension" ("date_id"), 
        CONSTRAINT m_villages_values_de_id_fk FOREIGN KEY (value_de_id) REFERENCES "m_villages_de" 
        ("dataelement_id"), 
        CONSTRAINT m_villages_values_location_id_fk FOREIGN KEY (value_location_id) REFERENCES 
        "m_villages_locations" ("location_id"), 
        CONSTRAINT m_villages_values_source_id_fk FOREIGN KEY (value_source_id) REFERENCES 
        "m_villages_de_value_source" ("value_source_id"), 
        UNIQUE (value_source_id, value_de_id, value_location_id) 
    );

-- Note: Additional tables for warehouse balance, movements, c_kobo_itns_team_main for ITNs campaigns data, districts etc., are implied but not listed here for brevity.
```
