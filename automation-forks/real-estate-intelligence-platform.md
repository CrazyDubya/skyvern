# Real Estate Intelligence Platform

## Overview
The Real Estate Intelligence Platform leverages Skyvern's automation capabilities to provide comprehensive property data aggregation, market analysis, investment scoring, and automated valuation for real estate agencies, property managers, and real estate investors.

## Core Features (Base Level)

### 1. Property Listing Aggregation
```python
from skyvern import Skyvern
from real_estate_platform.core.property_aggregator import PropertyAggregator

class PropertyListingAggregator:
    def __init__(self):
        self.skyvern = Skyvern()
        self.aggregator = PropertyAggregator()
        
    async def aggregate_property_listings(self, listing_sources):
        """Aggregate property listings from multiple real estate websites"""
        aggregated_listings = []
        
        for source in listing_sources:
            # Search for properties based on criteria
            search_task = await self.skyvern.run_task(
                prompt=f"""
                Search for properties on this real estate website with the following criteria:
                - Location: {source['search_criteria']['location']}
                - Property type: {source['search_criteria']['property_type']}
                - Price range: ${source['search_criteria']['min_price']} - ${source['search_criteria']['max_price']}
                - Bedrooms: {source['search_criteria']['bedrooms']}
                - Bathrooms: {source['search_criteria']['bathrooms']}
                
                Extract all property listings from the search results.
                """,
                url=source['base_url'],
                data_extraction_schema={
                    "type": "object",
                    "properties": {
                        "properties": {
                            "type": "array",
                            "items": {
                                "type": "object",
                                "properties": {
                                    "listing_id": {"type": "string"},
                                    "address": {"type": "string"},
                                    "city": {"type": "string"},
                                    "state": {"type": "string"},
                                    "zip_code": {"type": "string"},
                                    "price": {"type": "number"},
                                    "property_type": {"type": "string"},
                                    "bedrooms": {"type": "integer"},
                                    "bathrooms": {"type": "number"},
                                    "square_feet": {"type": "integer"},
                                    "lot_size": {"type": "number"},
                                    "year_built": {"type": "integer"},
                                    "listing_date": {"type": "string"},
                                    "days_on_market": {"type": "integer"},
                                    "agent_name": {"type": "string"},
                                    "agent_phone": {"type": "string"},
                                    "listing_url": {"type": "string"},
                                    "image_urls": {"type": "array", "items": {"type": "string"}},
                                    "description": {"type": "string"},
                                    "features": {"type": "array", "items": {"type": "string"}},
                                    "hoa_fees": {"type": "number"},
                                    "property_taxes": {"type": "number"}
                                }
                            }
                        }
                    }
                }
            )
            
            # Process and clean the listing data
            cleaned_listings = self.clean_listing_data(
                search_task.extracted_data['properties'], 
                source['name']
            )
            
            aggregated_listings.extend(cleaned_listings)
        
        # Remove duplicates and merge similar listings
        deduplicated_listings = self.deduplicate_listings(aggregated_listings)
        
        return deduplicated_listings
    
    def clean_listing_data(self, raw_listings, source_name):
        """Clean and standardize listing data"""
        cleaned_listings = []
        
        for listing in raw_listings:
            cleaned_listing = {
                **listing,
                "source": source_name,
                "price_per_sqft": listing['price'] / listing['square_feet'] if listing['square_feet'] > 0 else None,
                "extracted_at": pd.Timestamp.now(),
                "property_id": self.generate_property_id(listing),
                "standardized_address": self.standardize_address(listing['address'], listing['city'], listing['state'])
            }
            cleaned_listings.append(cleaned_listing)
        
        return cleaned_listings
```

### 2. Market Analysis Engine
```python
class MarketAnalysisEngine:
    def __init__(self, skyvern_client):
        self.skyvern = skyvern_client
        
    async def analyze_market_trends(self, market_areas):
        """Analyze market trends for specified areas"""
        market_analysis = []
        
        for area in market_areas:
            # Gather recent sales data
            sales_data = await self.gather_recent_sales_data(area)
            
            # Analyze price trends
            price_trends = await self.analyze_price_trends(area)
            
            # Get market statistics
            market_stats = await self.get_market_statistics(area)
            
            # Analyze inventory levels
            inventory_analysis = await self.analyze_inventory_levels(area)
            
            area_analysis = {
                "area": area['name'],
                "analysis_date": pd.Timestamp.now(),
                "sales_data": sales_data,
                "price_trends": price_trends,
                "market_statistics": market_stats,
                "inventory_analysis": inventory_analysis,
                "market_health_score": self.calculate_market_health_score(
                    price_trends, market_stats, inventory_analysis
                )
            }
            
            market_analysis.append(area_analysis)
        
        return market_analysis
    
    async def gather_recent_sales_data(self, area):
        """Gather recent sales data for market analysis"""
        sales_task = await self.skyvern.run_task(
            prompt=f"""
            Search for recently sold properties in {area['name']} 
            over the last {area['analysis_period']} months.
            Extract sale prices, sale dates, and property details.
            """,
            url=area['sales_data_url'],
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "recent_sales": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "address": {"type": "string"},
                                "sale_price": {"type": "number"},
                                "sale_date": {"type": "string"},
                                "original_list_price": {"type": "number"},
                                "days_on_market": {"type": "integer"},
                                "square_feet": {"type": "integer"},
                                "bedrooms": {"type": "integer"},
                                "bathrooms": {"type": "number"},
                                "year_built": {"type": "integer"},
                                "property_type": {"type": "string"}
                            }
                        }
                    }
                }
            }
        )
        
        return sales_task.extracted_data['recent_sales']
    
    async def analyze_price_trends(self, area):
        """Analyze price trends over time"""
        trends_task = await self.skyvern.run_task(
            prompt=f"""
            Look for price trend information and market reports for {area['name']}.
            Extract data about median prices, price changes over time,
            and market direction indicators.
            """,
            url=area['market_reports_url'],
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "median_price_current": {"type": "number"},
                    "median_price_previous_month": {"type": "number"},
                    "median_price_previous_year": {"type": "number"},
                    "price_change_month_over_month": {"type": "number"},
                    "price_change_year_over_year": {"type": "number"},
                    "average_days_on_market": {"type": "integer"},
                    "active_listings": {"type": "integer"},
                    "pending_sales": {"type": "integer"},
                    "closed_sales": {"type": "integer"},
                    "market_trend_direction": {"type": "string"}
                }
            }
        )
        
        return trends_task.extracted_data
```

### 3. Property Valuation System
```python
class PropertyValuationSystem:
    async def estimate_property_values(self, properties):
        """Estimate property values using comparative market analysis"""
        valuations = []
        
        for property_data in properties:
            # Find comparable properties
            comparables = await self.find_comparable_properties(property_data)
            
            # Perform comparative market analysis
            cma_result = self.perform_comparative_analysis(property_data, comparables)
            
            # Calculate estimated value
            estimated_value = self.calculate_estimated_value(cma_result)
            
            # Generate confidence score
            confidence_score = self.calculate_confidence_score(comparables, cma_result)
            
            valuation = {
                "property_id": property_data['property_id'],
                "estimated_value": estimated_value,
                "confidence_score": confidence_score,
                "valuation_date": pd.Timestamp.now(),
                "comparable_properties": comparables,
                "cma_analysis": cma_result,
                "value_range": {
                    "low": estimated_value * 0.9,
                    "high": estimated_value * 1.1
                }
            }
            
            valuations.append(valuation)
        
        return valuations
    
    async def find_comparable_properties(self, target_property):
        """Find comparable properties for valuation"""
        search_criteria = {
            "location_radius": 1.0,  # 1 mile radius
            "square_feet_variance": 0.2,  # 20% variance
            "age_variance": 10,  # 10 years
            "bedrooms": target_property['bedrooms'],
            "bathrooms_variance": 0.5
        }
        
        comparables_task = await self.skyvern.run_task(
            prompt=f"""
            Find recently sold properties comparable to:
            Address: {target_property['address']}
            Square feet: {target_property['square_feet']} (+/- 20%)
            Bedrooms: {target_property['bedrooms']}
            Bathrooms: {target_property['bathrooms']} (+/- 0.5)
            
            Look for properties sold in the last 6 months within 1 mile.
            """,
            url=f"https://realestate-comps.com/search",
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "comparable_properties": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "address": {"type": "string"},
                                "sale_price": {"type": "number"},
                                "sale_date": {"type": "string"},
                                "square_feet": {"type": "integer"},
                                "bedrooms": {"type": "integer"},
                                "bathrooms": {"type": "number"},
                                "year_built": {"type": "integer"},
                                "distance_miles": {"type": "number"},
                                "adjustments_needed": {"type": "array", "items": {"type": "string"}}
                            }
                        }
                    }
                }
            }
        )
        
        return comparables_task.extracted_data['comparable_properties']
```

## Level 1 Enhancements: Automated Valuation & Investment Scoring

### 1. Advanced Automated Valuation Model (AVM)
```python
from real_estate_platform.advanced.avm import AdvancedValuationModel
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor

class AdvancedAutomatedValuationModel:
    def __init__(self):
        self.valuation_models = self.load_valuation_models()
        self.feature_engineering = FeatureEngineering()
        
    async def generate_comprehensive_valuation(self, property_data):
        """Generate comprehensive property valuation using multiple models"""
        
        # Gather comprehensive property features
        comprehensive_features = await self.gather_comprehensive_features(property_data)
        
        # Apply multiple valuation models
        model_results = {}
        
        # Comparative Market Analysis Model
        model_results['cma_model'] = await self.apply_cma_model(comprehensive_features)
        
        # Machine Learning Model
        model_results['ml_model'] = self.apply_ml_model(comprehensive_features)
        
        # Cost Approach Model
        model_results['cost_model'] = await self.apply_cost_approach_model(comprehensive_features)
        
        # Income Approach Model (for investment properties)
        if comprehensive_features['property_type'] in ['rental', 'investment']:
            model_results['income_model'] = await self.apply_income_approach_model(comprehensive_features)
        
        # Ensemble the models for final valuation
        final_valuation = self.ensemble_valuations(model_results)
        
        return {
            "property_id": property_data['property_id'],
            "final_valuation": final_valuation,
            "model_results": model_results,
            "confidence_metrics": self.calculate_confidence_metrics(model_results),
            "valuation_breakdown": self.create_valuation_breakdown(model_results),
            "market_context": await self.add_market_context(property_data, final_valuation)
        }
    
    async def gather_comprehensive_features(self, property_data):
        """Gather comprehensive features for valuation"""
        features = property_data.copy()
        
        # Add location-based features
        location_features = await self.extract_location_features(property_data['address'])
        features.update(location_features)
        
        # Add market-based features
        market_features = await self.extract_market_features(property_data['zip_code'])
        features.update(market_features)
        
        # Add property-specific features
        property_features = await self.extract_property_features(property_data)
        features.update(property_features)
        
        return features
    
    async def extract_location_features(self, address):
        """Extract location-based features"""
        location_task = await self.skyvern.run_task(
            prompt=f"""
            Research the location features for {address}:
            - School district ratings
            - Crime rates
            - Walkability score
            - Public transportation access
            - Nearby amenities (shopping, restaurants, parks)
            - Distance to major employment centers
            - Future development plans
            """,
            url=f"https://location-intelligence.com/analyze?address={address}",
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "school_district_rating": {"type": "number"},
                    "crime_rate_index": {"type": "number"},
                    "walkability_score": {"type": "number"},
                    "public_transit_score": {"type": "number"},
                    "amenities_score": {"type": "number"},
                    "employment_center_distance": {"type": "number"},
                    "future_development_impact": {"type": "string"},
                    "neighborhood_appreciation_trend": {"type": "string"}
                }
            }
        )
        
        return location_task.extracted_data
    
    def apply_ml_model(self, features):
        """Apply machine learning model for valuation"""
        # Prepare feature vector
        feature_vector = self.feature_engineering.prepare_features(features)
        
        # Apply ensemble of ML models
        rf_prediction = self.valuation_models['random_forest'].predict([feature_vector])[0]
        gbm_prediction = self.valuation_models['gradient_boost'].predict([feature_vector])[0]
        neural_prediction = self.valuation_models['neural_network'].predict([feature_vector])[0]
        
        # Ensemble predictions
        ml_valuation = np.mean([rf_prediction, gbm_prediction, neural_prediction])
        
        return {
            "estimated_value": ml_valuation,
            "model_confidence": self.calculate_ml_confidence(feature_vector),
            "feature_importance": self.get_feature_importance(feature_vector),
            "individual_predictions": {
                "random_forest": rf_prediction,
                "gradient_boost": gbm_prediction,
                "neural_network": neural_prediction
            }
        }
```

### 2. Investment Analysis & Scoring System
```python
class InvestmentAnalysisEngine:
    def __init__(self):
        self.investment_models = self.load_investment_models()
        
    async def analyze_investment_opportunity(self, property_data, investment_criteria):
        """Analyze property as investment opportunity"""
        
        # Calculate financial metrics
        financial_metrics = await self.calculate_financial_metrics(property_data)
        
        # Analyze rental potential
        rental_analysis = await self.analyze_rental_potential(property_data)
        
        # Assess market dynamics
        market_dynamics = await self.assess_market_dynamics(property_data)
        
        # Calculate risk factors
        risk_assessment = await self.assess_investment_risks(property_data)
        
        # Generate investment score
        investment_score = self.calculate_investment_score(
            financial_metrics, rental_analysis, market_dynamics, risk_assessment
        )
        
        return {
            "property_id": property_data['property_id'],
            "investment_score": investment_score,
            "financial_metrics": financial_metrics,
            "rental_analysis": rental_analysis,
            "market_dynamics": market_dynamics,
            "risk_assessment": risk_assessment,
            "investment_recommendation": self.generate_investment_recommendation(investment_score),
            "sensitivity_analysis": await self.perform_sensitivity_analysis(property_data)
        }
    
    async def calculate_financial_metrics(self, property_data):
        """Calculate key financial metrics for investment analysis"""
        
        # Get rental income estimates
        rental_income = await self.estimate_rental_income(property_data)
        
        # Calculate operating expenses
        operating_expenses = await self.estimate_operating_expenses(property_data)
        
        # Get financing options
        financing_options = await self.get_financing_options(property_data)
        
        purchase_price = property_data['estimated_value']
        monthly_rental_income = rental_income['monthly_rent']
        annual_rental_income = monthly_rental_income * 12
        annual_expenses = operating_expenses['annual_total']
        net_operating_income = annual_rental_income - annual_expenses
        
        # Calculate key investment metrics
        metrics = {
            "purchase_price": purchase_price,
            "annual_rental_income": annual_rental_income,
            "annual_expenses": annual_expenses,
            "net_operating_income": net_operating_income,
            "cap_rate": (net_operating_income / purchase_price) * 100,
            "gross_yield": (annual_rental_income / purchase_price) * 100,
            "net_yield": (net_operating_income / purchase_price) * 100,
            "cash_on_cash_return": self.calculate_cash_on_cash_return(
                net_operating_income, financing_options, purchase_price
            ),
            "debt_service_coverage_ratio": self.calculate_dscr(
                net_operating_income, financing_options
            ),
            "break_even_ratio": (annual_expenses / annual_rental_income) * 100,
            "price_to_rent_ratio": purchase_price / annual_rental_income
        }
        
        return metrics
    
    async def analyze_rental_potential(self, property_data):
        """Analyze rental potential and market demand"""
        rental_task = await self.skyvern.run_task(
            prompt=f"""
            Research rental market for properties similar to:
            {property_data['address']} - {property_data['bedrooms']}BR/{property_data['bathrooms']}BA
            
            Find:
            - Current rental rates for similar properties
            - Vacancy rates in the area
            - Average time to rent
            - Rental demand indicators
            - Seasonal patterns
            """,
            url=f"https://rental-market-analysis.com/area/{property_data['zip_code']}",
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "market_rent_low": {"type": "number"},
                    "market_rent_high": {"type": "number"},
                    "market_rent_median": {"type": "number"},
                    "vacancy_rate": {"type": "number"},
                    "average_days_to_rent": {"type": "integer"},
                    "rental_demand_score": {"type": "number"},
                    "seasonal_variations": {"type": "array", "items": {"type": "object"}},
                    "comparable_rentals": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "address": {"type": "string"},
                                "monthly_rent": {"type": "number"},
                                "bedrooms": {"type": "integer"},
                                "bathrooms": {"type": "number"},
                                "square_feet": {"type": "integer"}
                            }
                        }
                    }
                }
            }
        )
        
        rental_data = rental_task.extracted_data
        
        # Calculate rental yield potential
        estimated_monthly_rent = rental_data['market_rent_median']
        rental_yield = (estimated_monthly_rent * 12 / property_data['estimated_value']) * 100
        
        return {
            **rental_data,
            "estimated_monthly_rent": estimated_monthly_rent,
            "rental_yield": rental_yield,
            "rent_to_value_ratio": estimated_monthly_rent / property_data['estimated_value'],
            "rental_potential_score": self.calculate_rental_potential_score(rental_data)
        }
    
    async def assess_market_dynamics(self, property_data):
        """Assess market dynamics affecting investment potential"""
        market_task = await self.skyvern.run_task(
            prompt=f"""
            Research market dynamics for {property_data['city']}, {property_data['state']}:
            - Population growth trends
            - Employment growth
            - Major employers and industries
            - Infrastructure developments
            - Zoning changes
            - New construction permits
            - Economic indicators
            """,
            url=f"https://market-dynamics.com/city/{property_data['city']}-{property_data['state']}",
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "population_growth_rate": {"type": "number"},
                    "employment_growth_rate": {"type": "number"},
                    "major_employers": {"type": "array", "items": {"type": "string"}},
                    "planned_developments": {"type": "array", "items": {"type": "string"}},
                    "new_construction_permits": {"type": "integer"},
                    "median_household_income": {"type": "number"},
                    "economic_diversity_score": {"type": "number"},
                    "market_appreciation_forecast": {"type": "number"}
                }
            }
        )
        
        return market_task.extracted_data
    
    def calculate_investment_score(self, financial_metrics, rental_analysis, market_dynamics, risk_assessment):
        """Calculate overall investment score"""
        
        # Weight different factors
        weights = {
            "financial": 0.35,
            "rental": 0.25,
            "market": 0.25,
            "risk": 0.15
        }
        
        # Score each category (0-100)
        financial_score = self.score_financial_metrics(financial_metrics)
        rental_score = rental_analysis['rental_potential_score']
        market_score = self.score_market_dynamics(market_dynamics)
        risk_score = 100 - risk_assessment['overall_risk_score']  # Invert risk
        
        # Calculate weighted score
        overall_score = (
            financial_score * weights["financial"] +
            rental_score * weights["rental"] +
            market_score * weights["market"] +
            risk_score * weights["risk"]
        )
        
        return {
            "overall_score": overall_score,
            "category_scores": {
                "financial": financial_score,
                "rental": rental_score,
                "market": market_score,
                "risk": risk_score
            },
            "investment_grade": self.determine_investment_grade(overall_score),
            "score_explanation": self.explain_investment_score(
                financial_score, rental_score, market_score, risk_score
            )
        }
```

## Real Estate Platform Workflow

### Comprehensive Real Estate Intelligence Workflow
```yaml
title: Real Estate Intelligence Platform - Market Analysis & Investment Scoring
description: Complete real estate automation with property analysis and investment evaluation
workflow_definition:
  parameters:
    - key: target_markets
      parameter_type: workflow
      workflow_parameter_type: array
    - key: investment_criteria
      parameter_type: workflow
      workflow_parameter_type: object
    - key: property_sources
      parameter_type: workflow
      workflow_parameter_type: array
      
  blocks:
    # Property listing aggregation
    - block_type: for_loop
      label: aggregate_listings
      loop_over: property_sources
      blocks:
        - block_type: task
          label: scrape_listings
          url: "{loop_item.search_url}"
          navigation_goal: "Search and extract all property listings matching criteria"
          data_extraction_schema:
            type: object
            properties:
              properties:
                type: array
                items:
                  type: object
                  properties:
                    address: {type: string}
                    price: {type: number}
                    bedrooms: {type: integer}
                    bathrooms: {type: number}
                    square_feet: {type: integer}
                    
    # Market analysis
    - block_type: for_loop
      label: analyze_markets
      loop_over: target_markets
      blocks:
        - block_type: task
          label: market_research
          url: "{loop_item.market_data_url}"
          navigation_goal: "Extract market trends, sales data, and economic indicators"
          
    # Property valuation
    - block_type: code
      label: value_properties
      code: |
        from real_estate_platform.avm import AdvancedValuationModel
        
        avm = AdvancedValuationModel()
        valuations = []
        
        for listing in context.previous_blocks['aggregate_listings'].output:
            valuation = await avm.generate_comprehensive_valuation(listing)
            valuations.append(valuation)
        
        return valuations
        
    # Investment analysis
    - block_type: code
      label: analyze_investments
      code: |
        from real_estate_platform.investment import InvestmentAnalysisEngine
        
        investment_engine = InvestmentAnalysisEngine()
        investment_analyses = []
        
        for property_data in context.previous_blocks['value_properties'].output:
            analysis = await investment_engine.analyze_investment_opportunity(
                property_data, investment_criteria
            )
            investment_analyses.append(analysis)
        
        return investment_analyses
        
    # Generate recommendations
    - block_type: code
      label: generate_recommendations
      code: |
        from real_estate_platform.recommendations import RecommendationEngine
        
        rec_engine = RecommendationEngine()
        recommendations = rec_engine.generate_investment_recommendations(
            properties=context.previous_blocks['analyze_investments'].output,
            market_data=context.previous_blocks['analyze_markets'].output,
            criteria=investment_criteria
        )
        
        return recommendations
        
    # Create comprehensive report
    - block_type: code
      label: create_report
      code: |
        from real_estate_platform.reporting import ReportGenerator
        
        report_gen = ReportGenerator()
        comprehensive_report = report_gen.create_market_investment_report(
            listings=context.previous_blocks['aggregate_listings'].output,
            market_analysis=context.previous_blocks['analyze_markets'].output,
            valuations=context.previous_blocks['value_properties'].output,
            investments=context.previous_blocks['analyze_investments'].output,
            recommendations=context.previous_blocks['generate_recommendations'].output
        )
        
        return comprehensive_report
        
    # Send notifications
    - block_type: send_email
      label: send_report
      to: ["investors@realestate.com", "agents@realestate.com"]
      subject: "Daily Real Estate Intelligence Report"
      body: |
        Real Estate Intelligence Summary:
        
        Properties analyzed: {previous_blocks.aggregate_listings.property_count}
        Markets evaluated: {previous_blocks.analyze_markets.market_count}
        Investment opportunities: {previous_blocks.generate_recommendations.opportunity_count}
        
        See attached comprehensive analysis report.
      attachments: ["{previous_blocks.create_report.report_file}"]
```

## Installation & Configuration

### Prerequisites
```bash
# Install base Skyvern
pip install skyvern

# Install Real Estate Platform
pip install real-estate-intelligence-platform

# Install analysis libraries
pip install pandas numpy scikit-learn geopy
```

### Configuration
```python
# config/real_estate_config.py
REAL_ESTATE_CONFIG = {
    "property_sources": [
        {
            "name": "Zillow",
            "base_url": "https://www.zillow.com",
            "search_endpoint": "/homes/for-sale"
        },
        {
            "name": "Realtor.com",
            "base_url": "https://www.realtor.com",
            "search_endpoint": "/realestateandhomes-search"
        }
    ],
    "valuation": {
        "models_enabled": ["cma", "ml", "cost_approach"],
        "confidence_threshold": 0.8,
        "feature_importance_analysis": True
    },
    "investment_analysis": {
        "min_cap_rate": 6.0,
        "max_price_to_rent_ratio": 15,
        "target_cash_on_cash_return": 8.0
    },
    "market_analysis": {
        "update_frequency": "daily",
        "trend_analysis_period": "12_months",
        "comparable_distance_radius": 1.0
    }
}
```

This Real Estate Intelligence Platform provides Level 1 enhancements with advanced automated valuation models and comprehensive investment analysis, making it a powerful tool for real estate professionals and investors to make data-driven decisions.