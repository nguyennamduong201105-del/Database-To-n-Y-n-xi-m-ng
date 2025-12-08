
CREATE TABLE users (
    user_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    username VARCHAR(100) UNIQUE NOT NULL,
    full_name VARCHAR(200),
    email VARCHAR(200) UNIQUE,
    role VARCHAR(50) NOT NULL,
    created_at DATETIME DEFAULT GETDATE()
);


CREATE TABLE warehouses (
    warehouse_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    name VARCHAR(200) NOT NULL,
    address VARCHAR(MAX),
    capacity_tons DECIMAL(18,2),
    is_outdoor BIT DEFAULT 0
);

CREATE TABLE locations (
    location_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    warehouse_id uniqueidentifier NOT NULL,
    code VARCHAR(50) NOT NULL,
    description VARCHAR(MAX),
    capacity_tons DECIMAL(18,2),
    CONSTRAINT FK_location_wh FOREIGN KEY (warehouse_id) REFERENCES warehouses(warehouse_id),
    CONSTRAINT UQ_location UNIQUE (warehouse_id, code)
);


CREATE TABLE products (
    product_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(300) NOT NULL,
    uom VARCHAR(20) DEFAULT 'TON',
    is_raw_material BIT DEFAULT 0,
    default_reorder_qty DECIMAL(18,2) DEFAULT 0
);

CREATE TABLE lots (
    lot_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    product_id uniqueidentifier NOT NULL,
    lot_number VARCHAR(100) NOT NULL,
    production_date DATE,
    expiry_date DATE,
    quantity DECIMAL(18,2),
    status VARCHAR(50),
    CONSTRAINT FK_lot_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);


CREATE TABLE stock_quants (
    quant_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    product_id uniqueidentifier NOT NULL,
    location_id uniqueidentifier NOT NULL,
    lot_id uniqueidentifier NULL,
    quantity DECIMAL(18,2) NOT NULL DEFAULT 0,
    last_updated DATETIME DEFAULT GETDATE(),
    CONSTRAINT FK_sq_prod FOREIGN KEY (product_id) REFERENCES products(product_id),
    CONSTRAINT FK_sq_loc FOREIGN KEY (location_id) REFERENCES locations(location_id),
    CONSTRAINT FK_sq_lot FOREIGN KEY (lot_id) REFERENCES lots(lot_id),
    CONSTRAINT UQ_stock_quant UNIQUE (product_id, location_id, lot_id)
);

CREATE TABLE stock_moves (
    move_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    product_id uniqueidentifier NOT NULL,
    from_location_id uniqueidentifier NULL,
    to_location_id uniqueidentifier NULL,
    quantity DECIMAL(18,2) NOT NULL,
    move_type VARCHAR(30) NOT NULL,
    moved_at DATETIME DEFAULT GETDATE(),
    related_doc VARCHAR(MAX),
    CONSTRAINT FK_sm_prod FOREIGN KEY (product_id) REFERENCES products(product_id),
    CONSTRAINT FK_sm_from FOREIGN KEY (from_location_id) REFERENCES locations(location_id),
    CONSTRAINT FK_sm_to FOREIGN KEY (to_location_id) REFERENCES locations(location_id)
);


CREATE TABLE suppliers (
    supplier_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    name VARCHAR(300) NOT NULL,
    contact VARCHAR(MAX),
    address VARCHAR(MAX),
    rating DECIMAL(5,2),
    lead_time_days INT
);

CREATE TABLE purchase_orders (
    po_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    po_number VARCHAR(100) UNIQUE NOT NULL,
    supplier_id uniqueidentifier,
    order_date DATE DEFAULT GETDATE(),
    expected_date DATE,
    status VARCHAR(50),
    total_amount DECIMAL(18,2),
    CONSTRAINT FK_po_supplier FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);

CREATE TABLE purchase_lines (
    poline_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    po_id uniqueidentifier NOT NULL,
    product_id uniqueidentifier NOT NULL,
    quantity DECIMAL(18,2) NOT NULL,
    unit_price DECIMAL(18,2),
    unit VARCHAR(20),
    CONSTRAINT FK_pl_po FOREIGN KEY (po_id) REFERENCES purchase_orders(po_id),
    CONSTRAINT FK_pl_prod FOREIGN KEY (product_id) REFERENCES products(product_id)
);


CREATE TABLE forecasts (
    forecast_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    product_id uniqueidentifier NOT NULL,
    period_start DATE,
    period_end DATE,
    forecast_qty DECIMAL(18,2),
    method VARCHAR(50),
    created_at DATETIME DEFAULT GETDATE(),
    CONSTRAINT FK_fc_prod FOREIGN KEY (product_id) REFERENCES products(product_id)
);


CREATE TABLE boms (
    bom_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    product_id uniqueidentifier NOT NULL,
    description VARCHAR(MAX),
    updated_at DATETIME DEFAULT GETDATE(),
    CONSTRAINT FK_bom_prod FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE bom_lines (
    bomline_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    bom_id uniqueidentifier NOT NULL,
    component_product_id uniqueidentifier NOT NULL,
    qty_per DECIMAL(18,2) NOT NULL,
    CONSTRAINT FK_bl_bom FOREIGN KEY (bom_id) REFERENCES boms(bom_id),
    CONSTRAINT FK_bl_prod FOREIGN KEY (component_product_id) REFERENCES products(product_id)
);

CREATE TABLE work_orders (
    wo_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    bom_id uniqueidentifier NOT NULL,
    start_date DATE,
    end_date DATE,
    status VARCHAR(50),
    quantity DECIMAL(18,2),
    CONSTRAINT FK_wo_bom FOREIGN KEY (bom_id) REFERENCES boms(bom_id)
);


CREATE TABLE equipment (
    equipment_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    name VARCHAR(200),
    code VARCHAR(100),
    location VARCHAR(200),
    last_service DATE
);

CREATE TABLE maint_requests (
    mr_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    equipment_id uniqueidentifier,
    requested_by uniqueidentifier,
    request_date DATE,
    priority VARCHAR(20),
    status VARCHAR(20),
    description VARCHAR(MAX),
    CONSTRAINT FK_mr_equip FOREIGN KEY (equipment_id) REFERENCES equipment(equipment_id),
    CONSTRAINT FK_mr_user FOREIGN KEY (requested_by) REFERENCES users(user_id)
);


CREATE TABLE projects (
    project_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    name VARCHAR(200),
    start_date DATE,
    end_date DATE,
    status VARCHAR(30)
);

CREATE TABLE tasks (
    task_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    project_id uniqueidentifier,
    name VARCHAR(200),
    assigned_to uniqueidentifier,
    due_date DATE,
    status VARCHAR(30),
    CONSTRAINT FK_task_proj FOREIGN KEY (project_id) REFERENCES projects(project_id),
    CONSTRAINT FK_task_user FOREIGN KEY (assigned_to) REFERENCES users(user_id)
);


CREATE TABLE sensors (
    sensor_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    serial VARCHAR(200) UNIQUE,
    type VARCHAR(50),
    unit VARCHAR(20),
    location_id uniqueidentifier,
    threshold_high DECIMAL(18,2),
    threshold_low DECIMAL(18,2),
    status VARCHAR(50),
    CONSTRAINT FK_sensor_loc FOREIGN KEY (location_id) REFERENCES locations(location_id)
);

CREATE TABLE sensor_reads (
    read_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    sensor_id uniqueidentifier NOT NULL,
    ts DATETIME DEFAULT GETDATE(),
    value DECIMAL(18,2) NOT NULL,
    CONSTRAINT FK_sr_sensor FOREIGN KEY (sensor_id) REFERENCES sensors(sensor_id)
);

CREATE TABLE alerts (
    alert_id uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    sensor_id uniqueidentifier,
    ts DATETIME DEFAULT GETDATE(),
    value DECIMAL(18,2),
    severity VARCHAR(20),
    status VARCHAR(20) DEFAULT 'OPEN',
    message VARCHAR(MAX),
    CONSTRAINT FK_alert_sensor FOREIGN KEY (sensor_id) REFERENCES sensors(sensor_id)
);
