# fuel_Tracker.sql
-- ==========================================
-- FuelWatch Naija: Human-Centered Fuel Tracker
-- Participant: Gabriel Olatunji Alabi | Ilorin Hub
-- Domain: Oil & Gas - Human Livelihood
-- Method: 3 Cities x 3 Fuel Types x 3 Years = 27 Records
-- Contact: 08038526605 | gaalabi1@gmail.com
-- GitHub: GitHub.com/gabriel-alabi
-- ==========================================


-- Create Table - With Ethics (No driver PII, only station public data)
CREATE TABLE fuel_prices (
    fuel_id SERIAL PRIMARY KEY,
    price_date DATE NOT NULL,
    city VARCHAR(50) NOT NULL,
    station_name VARCHAR(100) NOT NULL,
    fuel_type VARCHAR(20) NOT NULL,
    price_naira NUMERIC(12,2) NOT NULL,
    availability_status BOOLEAN NOT NULL,
    notes VARCHAR(100)
);

-- Insert 27 Records - 3 Cities (Ilorin, Lagos, Abuja) x 3 Fuels (PMS, Diesel, Kerosene) x 3 Years (2024-2026)

-- YEAR 2024 - Subsidy removal year price jumps to N630-N1200
INSERT INTO fuel_prices (price_date, city, station_name, fuel_type, price_naira, availability_status, notes) VALUES
('2024-01-15', 'Ilorin', 'BOVAS Ilorin', 'PMS', 650.00, TRUE, 'Available'),
('2024-01-15', 'Ilorin', 'BOVAS Ilorin', 'Diesel', 1200.00, TRUE, 'Available'),
('2024-01-15', 'Ilorin', 'BOVAS Ilorin', 'Kerosene', 1100.00, FALSE, 'Scarcity'),
('2024-01-15', 'Lagos', 'NNPC Lagos', 'PMS', 630.00, TRUE, 'Available'),
('2024-01-15', 'Lagos', 'NNPC Lagos', 'Diesel', 1150.00, TRUE, 'Available'),
('2024-01-15', 'Lagos', 'NNPC Lagos', 'Kerosene', 1050.00, TRUE, 'Available'),
('2024-01-15', 'Abuja', 'TotalEnergies Abuja', 'PMS', 640.00, FALSE, 'Queue 3hrs'),
('2024-01-15', 'Abuja', 'TotalEnergies Abuja', 'Diesel', 1180.00, TRUE, 'Available'),
('2024-01-15', 'Abuja', 'TotalEnergies Abuja', 'Kerosene', 1080.00, TRUE, 'Available'),

-- YEAR 2025 - Price increases
('2025-01-15', 'Ilorin', 'BOVAS Ilorin', 'PMS', 850.00, TRUE, 'Available'),
('2025-01-15', 'Ilorin', 'BOVAS Ilorin', 'Diesel', 1350.00, TRUE, 'Available'),
('2025-01-15', 'Ilorin', 'BOVAS Ilorin', 'Kerosene', 1250.00, TRUE, 'Available'),
('2025-01-15', 'Lagos', 'NNPC Lagos', 'PMS', 830.00, TRUE, 'Available'),
('2025-01-15', 'Lagos', 'NNPC Lagos', 'Diesel', 1300.00, FALSE, 'Scarcity'),
('2025-01-15', 'Lagos', 'NNPC Lagos', 'Kerosene', 1200.00, TRUE, 'Available'),
('2025-01-15', 'Abuja', 'TotalEnergies Abuja', 'PMS', 840.00, TRUE, 'Available'),
('2025-01-15', 'Abuja', 'TotalEnergies Abuja', 'Diesel', 1330.00, TRUE, 'Available'),
('2025-01-15', 'Abuja', 'TotalEnergies Abuja', 'Kerosene', 1230.00, FALSE, 'Black market N1500'),

-- YEAR 2026 - Current high prices
('2026-01-15', 'Ilorin', 'BOVAS Ilorin', 'PMS', 950.00, TRUE, 'Available - Cheapest'),
('2026-01-15', 'Ilorin', 'BOVAS Ilorin', 'Diesel', 1500.00, TRUE, 'Available'),
('2026-01-15', 'Ilorin', 'BOVAS Ilorin', 'Kerosene', 1400.00, TRUE, 'Available'),
('2026-01-15', 'Lagos', 'NNPC Lagos', 'PMS', 930.00, FALSE, 'Queue 5hrs - Scarcity alert'),
('2026-01-15', 'Lagos', 'NNPC Lagos', 'Diesel', 1450.00, TRUE, 'Available'),
('2026-01-15', 'Lagos', 'NNPC Lagos', 'Kerosene', 1350.00, TRUE, 'Available'),
('2026-01-15', 'Abuja', 'TotalEnergies Abuja', 'PMS', 940.00, TRUE, 'Available'),
('2026-01-15', 'Abuja', 'TotalEnergies Abuja', 'Diesel', 1480.00, TRUE, 'Available'),
('2026-01-15', 'Abuja', 'TotalEnergies Abuja', 'Kerosene', 1380.00, TRUE, 'Available');

-- ==========================================
-- ANALYSIS QUERIES FOR DASHBOARD (Use in Slide 4 & 5)
-- ==========================================

-- 1. Cheapest station for PMS in 2026 (Human decision: Okada rider saves money)
SELECT city, station_name, fuel_type, AVG(price_naira) AS average_price
FROM fuel_prices
WHERE EXTRACT(YEAR FROM price_date) = 2026 AND fuel_type = 'PMS' AND availability_status = TRUE
GROUP BY city, station_name, fuel_type
ORDER BY average_price ASC;

-- 2. Fuel inflation 2024 to 2026 - How much okada rider suffers (Human impact)
SELECT city, fuel_type,
       MAX(CASE WHEN EXTRACT(YEAR FROM price_date) = 2024 THEN price_naira END) AS price_2024,
       MAX(CASE WHEN EXTRACT(YEAR FROM price_date) = 2026 THEN price_naira END) AS price_2026,
       ROUND(((MAX(CASE WHEN EXTRACT(YEAR FROM price_date) = 2026 THEN price_naira END) - 
               MAX(CASE WHEN EXTRACT(YEAR FROM price_date) = 2024 THEN price_naira END)) / 
               MAX(CASE WHEN EXTRACT(YEAR FROM price_date) = 2024 THEN price_naira END) * 100), 2) AS inflation_percent
FROM fuel_prices
GROUP BY city, fuel_type
ORDER BY inflation_percent DESC;

-- 3. Scarcity Alert - Stations with no fuel (RED alert for dashboard)
SELECT price_date, city, station_name, fuel_type, availability_status
FROM fuel_prices
WHERE availability_status = FALSE
ORDER BY price_date DESC;

-- 4. Human Impact Query - How much extra okada rider pays if buys black market
SELECT city, fuel_type, price_naira AS official_price,
       (price_naira * 1.8) AS black_market_price,
       (price_naira * 1.8 - price_naira) AS extra_loss_per_liter
FROM fuel_prices
WHERE EXTRACT(YEAR FROM price_date) = 2026 AND fuel_type = 'PMS' AND city = 'Ilorin';
