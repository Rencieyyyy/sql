# CREATE DATABASE IF NOT EXISTS crime_report_system;
USE crime_report_system;

CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20),
    email_verified_at TIMESTAMP NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('citizen', 'officer', 'admin') DEFAULT 'citizen',
    profile_photo VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_role (role),
    INDEX idx_is_active (is_active)
);

CREATE TABLE crime_types (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    code VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    severity_level ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
    icon VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_code (code),
    INDEX idx_severity (severity_level),
    INDEX idx_is_active (is_active)
);

CREATE TABLE locations (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    latitude DECIMAL(10,8) NOT NULL,
    longitude DECIMAL(11,8) NOT NULL,
    address VARCHAR(500),
    city VARCHAR(255),
    state VARCHAR(255),
    postal_code VARCHAR(20),
    country VARCHAR(255) DEFAULT 'Philippines',
    landmark VARCHAR(255),
    google_place_data JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_coordinates (latitude, longitude),
    INDEX idx_city (city),
    INDEX idx_postal_code (postal_code)
);

CREATE TABLE crime_reports (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    location_id BIGINT UNSIGNED NOT NULL,
    crime_type_id BIGINT UNSIGNED NOT NULL,
    assigned_officer_id BIGINT UNSIGNED NULL,
    incident_number VARCHAR(50) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    severity ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
    status ENUM('pending', 'investigating', 'resolved', 'closed') DEFAULT 'pending',
    incident_datetime DATETIME NOT NULL,
    is_anonymous BOOLEAN DEFAULT FALSE,
    witness_info JSON,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (location_id) REFERENCES locations(id) ON DELETE CASCADE,
    FOREIGN KEY (crime_type_id) REFERENCES crime_types(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_officer_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_incident_number (incident_number),
    INDEX idx_status (status),
    INDEX idx_severity (severity),
    INDEX idx_incident_datetime (incident_datetime),
    INDEX idx_user_id (user_id),
    INDEX idx_assigned_officer (assigned_officer_id)
);

CREATE TABLE crime_evidence (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    crime_report_id BIGINT UNSIGNED NOT NULL,
    evidence_type ENUM('photo', 'video', 'audio', 'document') NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    original_filename VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size BIGINT UNSIGNED NOT NULL,
    description TEXT,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (crime_report_id) REFERENCES crime_reports(id) ON DELETE CASCADE,
    INDEX idx_crime_report_id (crime_report_id),
    INDEX idx_evidence_type (evidence_type)
);

CREATE TABLE safe_routes (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    origin_location_id BIGINT UNSIGNED NOT NULL,
    destination_location_id BIGINT UNSIGNED NOT NULL,
    route_data JSON NOT NULL,
    total_distance DECIMAL(10,2),
    estimated_duration INTEGER,
    safety_score DECIMAL(3,2) DEFAULT 0.00,
    crime_incidents_nearby JSON,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (origin_location_id) REFERENCES locations(id) ON DELETE CASCADE,
    FOREIGN KEY (destination_location_id) REFERENCES locations(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_safety_score (safety_score),
    INDEX idx_is_active (is_active)
);

CREATE TABLE crime_hotspots (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    location_id BIGINT UNSIGNED NOT NULL,
    crime_type_id BIGINT UNSIGNED NOT NULL,
    incident_count INTEGER DEFAULT 0,
    danger_level DECIMAL(3,2) DEFAULT 0.00,
    radius_meters DECIMAL(10,2) DEFAULT 500.00,
    start_date DATE NOT NULL,
    end_date DATE,
    statistics JSON,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (location_id) REFERENCES locations(id) ON DELETE CASCADE,
    FOREIGN KEY (crime_type_id) REFERENCES crime_types(id) ON DELETE CASCADE,
    INDEX idx_location_id (location_id),
    INDEX idx_crime_type_id (crime_type_id),
    INDEX idx_danger_level (danger_level),
    INDEX idx_date_range (start_date, end_date),
    INDEX idx_is_active (is_active)
);

CREATE TABLE notifications (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    crime_report_id BIGINT UNSIGNED NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    type ENUM('crime_alert', 'route_warning', 'status_update', 'system') DEFAULT 'system',
    priority ENUM('low', 'medium', 'high', 'urgent') DEFAULT 'medium',
    is_read BOOLEAN DEFAULT FALSE,
    data JSON,
    read_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (crime_report_id) REFERENCES crime_reports(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_is_read (is_read),
    INDEX idx_type (type),
    INDEX idx_priority (priority),
    INDEX idx_created_at (created_at)
);

CREATE TABLE emergency_contacts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20) NOT NULL,
    relationship VARCHAR(100),
    is_primary BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_is_primary (is_primary),
    INDEX idx_is_active (is_active)
);

CREATE TABLE user_locations (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    location_id BIGINT UNSIGNED NOT NULL,
    location_type ENUM('home', 'work', 'frequent', 'emergency') DEFAULT 'frequent',
    label VARCHAR(100),
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (location_id) REFERENCES locations(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_location_type (location_type),
    INDEX idx_is_default (is_default)
);

CREATE TABLE route_feedback (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    safe_route_id BIGINT UNSIGNED NOT NULL,
    safety_rating INTEGER CHECK (safety_rating >= 1 AND safety_rating <= 5),
    feedback TEXT,
    route_taken BOOLEAN DEFAULT FALSE,
    incident_encountered JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (safe_route_id) REFERENCES safe_routes(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_safe_route_id (safe_route_id),
    INDEX idx_safety_rating (safety_rating)
);

CREATE TABLE google_maps_cache (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    cache_key VARCHAR(255) UNIQUE NOT NULL,
    response_data JSON NOT NULL,
    request_type ENUM('geocoding', 'directions', 'places', 'distance_matrix') NOT NULL,
    ttl_seconds INTEGER DEFAULT 3600,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_cache_key (cache_key),
    INDEX idx_expires_at (expires_at),
    INDEX idx_request_type (request_type)
);

CREATE TABLE system_settings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    `key` VARCHAR(255) UNIQUE NOT NULL,
    `value` TEXT NOT NULL,
    `type` ENUM('string', 'integer', 'boolean', 'json') DEFAULT 'string',
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_key (`key`)
);

CREATE TABLE api_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NULL,
    endpoint VARCHAR(255) NOT NULL,
    method VARCHAR(10) NOT NULL,
    request_data JSON,
    response_data JSON,
    status_code INTEGER NOT NULL,
    response_time DECIMAL(8,3),
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_user_id (user_id),
    INDEX idx_endpoint (endpoint),
    INDEX idx_status_code (status_code),
    INDEX idx_created_at (created_at)
);

INSERT INTO crime_types (name, code, description, severity_level, is_active) VALUES
('Theft', 'THEFT', 'Unlawful taking of property', 'medium', TRUE),
('Robbery', 'ROBBERY', 'Taking property by force or threat', 'high', TRUE),
('Assault', 'ASSAULT', 'Physical attack on another person', 'high', TRUE),
('Burglary', 'BURGLARY', 'Breaking and entering with intent to steal', 'high', TRUE),
('Vandalism', 'VANDALISM', 'Destruction of property', 'low', TRUE),
('Drug Related', 'DRUG', 'Illegal drug activities', 'medium', TRUE),
('Fraud', 'FRAUD', 'Deceptive practices for financial gain', 'medium', TRUE),
('Murder', 'MURDER', 'Unlawful killing of another person', 'critical', TRUE),
('Kidnapping', 'KIDNAP', 'Unlawful restraint or abduction', 'critical', TRUE),
('Sexual Assault', 'SEX_ASSAULT', 'Non-consensual sexual contact', 'critical', TRUE);

INSERT INTO system_settings (`key`, `value`, `type`, description) VALUES
('google_maps_api_key', '', 'string', 'Google Maps API key for geocoding and routing'),
('max_file_upload_size', '10485760', 'integer', 'Maximum file upload size in bytes (10MB)'),
('cache_ttl_hours', '24', 'integer', 'Default cache TTL in hours'),
('safety_radius_meters', '1000', 'integer', 'Default radius for safety calculations'),
('notification_enabled', 'true', 'boolean', 'Enable push notifications'),
('anonymous_reporting', 'true', 'boolean', 'Allow anonymous crime reporting'),
('auto_assign_officers', 'false', 'boolean', 'Automatically assign officers to reports');

INSERT INTO users (name, email, password, role, is_active) VALUES
('System Administrator', 'admin@crimereport.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'admin', TRUE);

CREATE INDEX idx_crime_reports_composite ON crime_reports(status, severity, incident_datetime);
CREATE INDEX idx_locations_composite ON locations(city, state, country);
CREATE INDEX idx_notifications_unread ON notifications(user_id, is_read, created_at);
CREATE INDEX idx_hotspots_active ON crime_hotspots(is_active, danger_level, location_id);
