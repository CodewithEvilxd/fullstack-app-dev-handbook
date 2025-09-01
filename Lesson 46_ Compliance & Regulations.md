# Lesson 46: Compliance & Regulations

## 🎯 **Learning Objectives**
- Understand major compliance frameworks and regulations
- Implement GDPR, CCPA, and HIPAA compliance features
- Handle data privacy and user consent management
- Create audit trails and compliance reporting
- Navigate international data protection laws

## 📚 **Table of Contents**
1. [Compliance Fundamentals](#compliance-fundamentals)
2. [GDPR Compliance](#gdpr-compliance)
3. [CCPA Compliance](#ccpa-compliance)
4. [HIPAA Compliance](#hipaa-compliance)
5. [Data Privacy Management](#data-privacy-management)
6. [Consent Management](#consent-management)
7. [Audit & Logging](#audit--logging)
8. [International Compliance](#international-compliance)
9. [Compliance Automation](#compliance-automation)
10. [Practical Examples](#practical-examples)

---

## 📋 **Compliance Fundamentals**

### **Major Compliance Frameworks**
- **GDPR**: General Data Protection Regulation (EU)
- **CCPA**: California Consumer Privacy Act (US)
- **HIPAA**: Health Insurance Portability and Accountability Act (US)
- **PCI DSS**: Payment Card Industry Data Security Standard
- **SOX**: Sarbanes-Oxley Act (Financial reporting)
- **ISO 27001**: Information Security Management

### **Key Compliance Principles**
- **Data Minimization**: Collect only necessary data
- **Purpose Limitation**: Use data only for intended purposes
- **Storage Limitation**: Keep data only as long as necessary
- **Data Accuracy**: Ensure data accuracy and update when needed
- **Integrity & Confidentiality**: Protect data from unauthorized access
- **Accountability**: Be able to demonstrate compliance

### **Compliance Lifecycle**
1. **Assessment**: Identify applicable regulations
2. **Gap Analysis**: Compare current practices with requirements
3. **Implementation**: Develop and implement compliance measures
4. **Monitoring**: Continuous monitoring and auditing
5. **Reporting**: Regular compliance reporting
6. **Remediation**: Address identified issues

---

## 🇪🇺 **GDPR Compliance**

### **GDPR Key Requirements**
- **Lawful Basis**: Legal grounds for processing personal data
- **Data Subject Rights**: Rights of individuals over their data
- **Data Protection Officer**: Required for certain organizations
- **Data Protection Impact Assessment**: Required for high-risk processing
- **Breach Notification**: Must notify authorities within 72 hours
- **Privacy by Design**: Privacy considerations built into systems

### **GDPR Implementation**
```javascript
// services/gdprCompliance.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { Alert, Linking } from 'react-native';

class GDPRCompliance {
  constructor() {
    this.consentVersion = '1.0';
    this.consentKey = 'gdpr_consent';
  }

  // Check if user is in GDPR jurisdiction
  async isInGDPRJurisdiction() {
    // This is a simplified check - in production, use IP geolocation
    const userLocation = await this.getUserLocation();

    // EU countries
    const euCountries = [
      'AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR',
      'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL',
      'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB'
    ];

    return euCountries.includes(userLocation.countryCode);
  }

  // Get user consent status
  async getConsentStatus() {
    try {
      const consentData = await AsyncStorage.getItem(this.consentKey);
      return consentData ? JSON.parse(consentData) : null;
    } catch (error) {
      console.error('Error getting consent status:', error);
      return null;
    }
  }

  // Request user consent
  async requestConsent() {
    return new Promise((resolve) => {
      Alert.alert(
        'Privacy Notice',
        'We collect and process your personal data to provide our services. ' +
        'You can manage your privacy preferences at any time.',
        [
          {
            text: 'Manage Preferences',
            onPress: () => {
              this.showConsentPreferences().then(resolve);
            },
          },
          {
            text: 'Accept All',
            onPress: () => {
              this.grantConsent({
                necessary: true,
                analytics: true,
                marketing: true,
                functional: true,
              }).then(resolve);
            },
          },
          {
            text: 'Reject All',
            onPress: () => {
              this.grantConsent({
                necessary: true,
                analytics: false,
                marketing: false,
                functional: false,
              }).then(resolve);
            },
          },
        ],
        { cancelable: false }
      );
    });
  }

  // Show detailed consent preferences
  async showConsentPreferences() {
    // This would typically open a detailed preferences screen
    // For now, we'll use alerts for demonstration
    return new Promise((resolve) => {
      Alert.alert(
        'Cookie Preferences',
        'Choose which types of cookies and data processing you consent to:',
        [
          {
            text: 'Essential Only',
            onPress: () => {
              this.grantConsent({
                necessary: true,
                analytics: false,
                marketing: false,
                functional: false,
              }).then(resolve);
            },
          },
          {
            text: 'Analytics Only',
            onPress: () => {
              this.grantConsent({
                necessary: true,
                analytics: true,
                marketing: false,
                functional: false,
              }).then(resolve);
            },
          },
          {
            text: 'All Categories',
            onPress: () => {
              this.grantConsent({
                necessary: true,
                analytics: true,
                marketing: true,
                functional: true,
              }).then(resolve);
            },
          },
        ]
      );
    });
  }

  // Grant user consent
  async grantConsent(consent) {
    const consentData = {
      version: this.consentVersion,
      timestamp: new Date().toISOString(),
      consent,
      userId: await this.getCurrentUserId(),
    };

    try {
      await AsyncStorage.setItem(this.consentKey, JSON.stringify(consentData));

      // Log consent event
      await this.logConsentEvent('consent_granted', consentData);

      return consentData;
    } catch (error) {
      console.error('Error saving consent:', error);
      throw error;
    }
  }

  // Withdraw consent
  async withdrawConsent() {
    try {
      const consentData = await this.getConsentStatus();
      if (consentData) {
        consentData.withdrawnAt = new Date().toISOString();
        consentData.consent = {
          necessary: false,
          analytics: false,
          marketing: false,
          functional: false,
        };

        await AsyncStorage.setItem(this.consentKey, JSON.stringify(consentData));

        // Log withdrawal event
        await this.logConsentEvent('consent_withdrawn', consentData);

        // Delete user data if requested
        await this.deleteUserData();
      }
    } catch (error) {
      console.error('Error withdrawing consent:', error);
      throw error;
    }
  }

  // Check if consent is valid
  async isConsentValid() {
    const consentData = await this.getConsentStatus();

    if (!consentData) return false;

    // Check if consent version is current
    if (consentData.version !== this.consentVersion) {
      return false;
    }

    // Check if consent has been withdrawn
    if (consentData.withdrawnAt) {
      return false;
    }

    // Check if consent is expired (GDPR requires periodic renewal)
    const consentDate = new Date(consentData.timestamp);
    const oneYearAgo = new Date();
    oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

    if (consentDate < oneYearAgo) {
      return false;
    }

    return true;
  }

  // Handle data subject access request (DSAR)
  async handleDataAccessRequest() {
    const userId = await this.getCurrentUserId();
    const userData = await this.collectUserData(userId);

    // In a real implementation, this would generate a report
    // and send it to the user via email
    console.log('Data access request for user:', userId);
    console.log('Collected data:', userData);

    return {
      userId,
      data: userData,
      timestamp: new Date().toISOString(),
    };
  }

  // Handle data deletion request
  async handleDataDeletionRequest() {
    const userId = await this.getCurrentUserId();

    // Delete user data from all systems
    await this.deleteUserData(userId);

    // Log deletion event
    await this.logComplianceEvent('data_deletion', {
      userId,
      timestamp: new Date().toISOString(),
    });

    return {
      userId,
      deleted: true,
      timestamp: new Date().toISOString(),
    };
  }

  // Collect all user data
  async collectUserData(userId) {
    // This would collect data from all systems
    const data = {
      profile: await this.getUserProfile(userId),
      activity: await this.getUserActivity(userId),
      preferences: await this.getUserPreferences(userId),
      consent: await this.getConsentStatus(),
    };

    return data;
  }

  // Delete user data
  async deleteUserData(userId) {
    // Delete from all data stores
    await this.deleteUserProfile(userId);
    await this.deleteUserActivity(userId);
    await this.deleteUserPreferences(userId);

    // Anonymize remaining data
    await this.anonymizeUserData(userId);
  }

  // Log compliance events
  async logComplianceEvent(eventType, data) {
    const event = {
      type: eventType,
      data,
      timestamp: new Date().toISOString(),
      ipAddress: await this.getClientIP(),
      userAgent: 'React Native App',
    };

    // Send to compliance logging service
    console.log('Compliance event:', event);
  }

  // Log consent events
  async logConsentEvent(eventType, consentData) {
    await this.logComplianceEvent(eventType, {
      consent: consentData.consent,
      timestamp: consentData.timestamp,
      version: consentData.version,
    });
  }

  // Helper methods (simplified implementations)
  async getUserLocation() {
    // In production, use IP geolocation service
    return { countryCode: 'US' };
  }

  async getCurrentUserId() {
    // Get from auth context
    return 'user123';
  }

  async getUserProfile(userId) {
    return { name: 'John Doe', email: 'john@example.com' };
  }

  async getUserActivity(userId) {
    return [{ action: 'login', timestamp: new Date().toISOString() }];
  }

  async getUserPreferences(userId) {
    return { theme: 'dark', notifications: true };
  }

  async deleteUserProfile(userId) {
    console.log('Deleting user profile for:', userId);
  }

  async deleteUserActivity(userId) {
    console.log('Deleting user activity for:', userId);
  }

  async deleteUserPreferences(userId) {
    console.log('Deleting user preferences for:', userId);
  }

  async anonymizeUserData(userId) {
    console.log('Anonymizing remaining data for:', userId);
  }

  async getClientIP() {
    // In production, get from server
    return '192.168.1.1';
  }
}

export default new GDPRCompliance();
```

### **Data Processing Agreement**
```javascript
// services/dataProcessing.js
class DataProcessingAgreement {
  constructor() {
    this.processingActivities = new Map();
  }

  // Register data processing activity
  registerProcessingActivity(activity) {
    const processingActivity = {
      id: activity.id,
      name: activity.name,
      purpose: activity.purpose,
      legalBasis: activity.legalBasis,
      dataCategories: activity.dataCategories,
      recipients: activity.recipients,
      retentionPeriod: activity.retentionPeriod,
      securityMeasures: activity.securityMeasures,
      registeredAt: new Date().toISOString(),
    };

    this.processingActivities.set(activity.id, processingActivity);

    // Log registration
    this.logProcessingActivity('registered', processingActivity);
  }

  // Update processing activity
  updateProcessingActivity(id, updates) {
    const activity = this.processingActivities.get(id);
    if (activity) {
      const updatedActivity = {
        ...activity,
        ...updates,
        updatedAt: new Date().toISOString(),
      };

      this.processingActivities.set(id, updatedActivity);
      this.logProcessingActivity('updated', updatedActivity);
    }
  }

  // Get processing activity
  getProcessingActivity(id) {
    return this.processingActivities.get(id);
  }

  // List all processing activities
  getAllProcessingActivities() {
    return Array.from(this.processingActivities.values());
  }

  // Check if processing activity is compliant
  isProcessingActivityCompliant(id) {
    const activity = this.processingActivities.get(id);
    if (!activity) return false;

    // Check legal basis
    const validLegalBases = [
      'consent',
      'contract',
      'legal_obligation',
      'vital_interests',
      'public_task',
      'legitimate_interests',
    ];

    if (!validLegalBases.includes(activity.legalBasis)) {
      return false;
    }

    // Check retention period
    if (!activity.retentionPeriod || activity.retentionPeriod < 1) {
      return false;
    }

    // Check security measures
    const requiredMeasures = ['encryption', 'access_control', 'audit_logging'];
    const hasRequiredMeasures = requiredMeasures.every(measure =>
      activity.securityMeasures.includes(measure)
    );

    if (!hasRequiredMeasures) {
      return false;
    }

    return true;
  }

  // Generate DPIA (Data Protection Impact Assessment)
  generateDPIA(id) {
    const activity = this.processingActivities.get(id);
    if (!activity) return null;

    const dpia = {
      processingActivity: activity.name,
      purpose: activity.purpose,
      dataCategories: activity.dataCategories,
      riskAssessment: this.assessRisk(activity),
      mitigationMeasures: this.getMitigationMeasures(activity),
      generatedAt: new Date().toISOString(),
    };

    return dpia;
  }

  // Assess risk level
  assessRisk(activity) {
    let riskScore = 0;

    // High risk if processing sensitive data
    if (activity.dataCategories.includes('health') ||
        activity.dataCategories.includes('biometric')) {
      riskScore += 3;
    }

    // High risk if large scale processing
    if (activity.recipients.length > 10) {
      riskScore += 2;
    }

    // High risk if long retention period
    if (activity.retentionPeriod > 365) {
      riskScore += 2;
    }

    if (riskScore >= 5) return 'High';
    if (riskScore >= 3) return 'Medium';
    return 'Low';
  }

  // Get mitigation measures
  getMitigationMeasures(activity) {
    const measures = [];

    if (activity.dataCategories.includes('personal')) {
      measures.push('Data minimization');
      measures.push('Purpose limitation');
      measures.push('Storage limitation');
    }

    if (activity.recipients.length > 0) {
      measures.push('Data processing agreements with recipients');
      measures.push('Regular recipient audits');
    }

    measures.push('Encryption at rest and in transit');
    measures.push('Regular security assessments');
    measures.push('Incident response plan');

    return measures;
  }

  // Log processing activity events
  logProcessingActivity(eventType, activity) {
    const logEntry = {
      event: eventType,
      activityId: activity.id,
      activityName: activity.name,
      timestamp: new Date().toISOString(),
    };

    console.log('Processing activity log:', logEntry);
  }

  // Generate compliance report
  generateComplianceReport() {
    const activities = this.getAllProcessingActivities();
    const report = {
      totalActivities: activities.length,
      compliantActivities: activities.filter(a => this.isProcessingActivityCompliant(a.id)).length,
      nonCompliantActivities: activities.filter(a => !this.isProcessingActivityCompliant(a.id)).length,
      highRiskActivities: activities.filter(a => this.assessRisk(a) === 'High').length,
      generatedAt: new Date().toISOString(),
      activities: activities.map(activity => ({
        id: activity.id,
        name: activity.name,
        compliant: this.isProcessingActivityCompliant(activity.id),
        risk: this.assessRisk(activity),
      })),
    };

    return report;
  }
}

export default new DataProcessingAgreement();
```

---

## 🇺🇸 **CCPA Compliance**

### **CCPA Key Requirements**
- **Right to Know**: What personal information is collected
- **Right to Delete**: Delete personal information
- **Right to Opt-Out**: Opt-out of sale of personal information
- **Right to Non-Discrimination**: No discrimination for exercising rights
- **Data Minimization**: Collect only necessary information
- **Security**: Protect personal information

### **CCPA Implementation**
```javascript
// services/ccpaCompliance.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { Alert, Linking } from 'react-native';

class CCPACompliance {
  constructor() {
    this.privacyNoticeVersion = '1.0';
    this.privacyKey = 'ccpa_privacy';
  }

  // Check if user is California resident
  async isCaliforniaResident() {
    // This is a simplified check - in production, use IP geolocation
    const userLocation = await this.getUserLocation();
    return userLocation.stateCode === 'CA';
  }

  // Show CCPA privacy notice
  async showPrivacyNotice() {
    const hasSeenNotice = await this.hasSeenPrivacyNotice();

    if (!hasSeenNotice) {
      Alert.alert(
        'California Privacy Notice',
        'We collect personal information as described in our Privacy Policy. ' +
        'California residents have additional rights under CCPA.',
        [
          {
            text: 'Learn More',
            onPress: () => this.showDetailedPrivacyInfo(),
          },
          {
            text: 'Accept',
            onPress: () => this.acceptPrivacyNotice(),
          },
        ],
        { cancelable: false }
      );
    }
  }

  // Show detailed privacy information
  async showDetailedPrivacyInfo() {
    Alert.alert(
      'Your CCPA Rights',
      'As a California resident, you have the right to:\n\n' +
      '• Know what personal information we collect\n' +
      '• Know if we sell your personal information\n' +
      '• Opt-out of the sale of your personal information\n' +
      '• Request deletion of your personal information\n' +
      '• Receive equal service regardless of privacy choices',
      [
        {
          text: 'Privacy Policy',
          onPress: () => Linking.openURL('https://example.com/privacy'),
        },
        {
          text: 'Accept',
          onPress: () => this.acceptPrivacyNotice(),
        },
      ]
    );
  }

  // Accept privacy notice
  async acceptPrivacyNotice() {
    const privacyData = {
      version: this.privacyNoticeVersion,
      acceptedAt: new Date().toISOString(),
      userId: await this.getCurrentUserId(),
    };

    await AsyncStorage.setItem(this.privacyKey, JSON.stringify(privacyData));
  }

  // Check if user has seen privacy notice
  async hasSeenPrivacyNotice() {
    const privacyData = await AsyncStorage.getItem(this.privacyKey);
    return privacyData !== null;
  }

  // Handle "Do Not Sell My Personal Information" request
  async handleDoNotSellRequest() {
    const userId = await this.getCurrentUserId();

    // Update user's sale opt-out status
    await this.updateSaleOptOutStatus(userId, true);

    // Stop selling user's data
    await this.stopSellingUserData(userId);

    // Log the request
    await this.logCCPAEvent('do_not_sell_request', { userId });

    Alert.alert(
      'Request Processed',
      'We have processed your request to not sell your personal information. ' +
      'This may take up to 45 days to fully implement.'
    );
  }

  // Handle data deletion request
  async handleDeletionRequest() {
    const userId = await this.getCurrentUserId();

    Alert.alert(
      'Confirm Deletion',
      'Are you sure you want to delete all your personal information? ' +
      'This action cannot be undone.',
      [
        {
          text: 'Cancel',
          style: 'cancel',
        },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: () => this.processDeletionRequest(userId),
        },
      ]
    );
  }

  // Process deletion request
  async processDeletionRequest(userId) {
    try {
      // Delete user data
      await this.deleteUserData(userId);

      // Log the deletion
      await this.logCCPAEvent('data_deletion', { userId });

      // Sign out user
      await this.signOutUser();

      Alert.alert(
        'Data Deleted',
        'Your personal information has been deleted from our systems.'
      );
    } catch (error) {
      console.error('Deletion error:', error);
      Alert.alert(
        'Error',
        'There was an error processing your deletion request. Please try again.'
      );
    }
  }

  // Handle "Know My Data" request
  async handleKnowMyDataRequest() {
    const userId = await this.getCurrentUserId();

    try {
      const userData = await this.collectUserData(userId);

      // In a real app, this would generate a report and send it to the user
      const dataSummary = {
        categories: Object.keys(userData),
        collectionDates: {
          from: userData.profile?.createdAt,
          to: new Date().toISOString(),
        },
        sources: ['Direct input', 'App usage', 'Third parties'],
      };

      Alert.alert(
        'Your Data Summary',
        `We have collected data in these categories:\n\n${dataSummary.categories.join('\n')}\n\n` +
        'A detailed report has been sent to your email address.'
      );

      // Log the request
      await this.logCCPAEvent('know_my_data_request', { userId, dataSummary });

    } catch (error) {
      console.error('Data collection error:', error);
      Alert.alert(
        'Error',
        'There was an error retrieving your data. Please try again.'
      );
    }
  }

  // Update sale opt-out status
  async updateSaleOptOutStatus(userId, optOut) {
    // Update user preferences
    const preferences = await this.getUserPreferences(userId);
    preferences.ccpaSaleOptOut = optOut;
    await this.saveUserPreferences(userId, preferences);
  }

  // Stop selling user data
  async stopSellingUserData(userId) {
    // Remove user from marketing lists
    await this.removeFromMarketingLists(userId);

    // Stop sharing with third parties for marketing
    await this.stopThirdPartySharing(userId);

    // Update data processing agreements
    await this.updateDataProcessingAgreements(userId);
  }

  // Collect user data for reporting
  async collectUserData(userId) {
    const data = {
      profile: await this.getUserProfile(userId),
      activity: await this.getUserActivity(userId),
      preferences: await this.getUserPreferences(userId),
      marketing: await this.getMarketingData(userId),
    };

    return data;
  }

  // Delete user data
  async deleteUserData(userId) {
    await this.deleteUserProfile(userId);
    await this.deleteUserActivity(userId);
    await this.deleteUserPreferences(userId);
    await this.deleteMarketingData(userId);
  }

  // Log CCPA events
  async logCCPAEvent(eventType, data) {
    const event = {
      type: eventType,
      data,
      timestamp: new Date().toISOString(),
      regulation: 'CCPA',
    };

    console.log('CCPA event:', event);
    // Send to compliance logging service
  }

  // Helper methods (simplified implementations)
  async getUserLocation() {
    return { stateCode: 'CA' }; // Assume California for demo
  }

  async getCurrentUserId() {
    return 'user123';
  }

  async getUserProfile(userId) {
    return { name: 'John Doe', email: 'john@example.com' };
  }

  async getUserActivity(userId) {
    return [{ action: 'login', timestamp: new Date().toISOString() }];
  }

  async getUserPreferences(userId) {
    return { theme: 'dark', notifications: true };
  }

  async getMarketingData(userId) {
    return { interests: ['technology', 'sports'] };
  }

  async saveUserPreferences(userId, preferences) {
    console.log('Saving preferences for:', userId, preferences);
  }

  async removeFromMarketingLists(userId) {
    console.log('Removing from marketing lists:', userId);
  }

  async stopThirdPartySharing(userId) {
    console.log('Stopping third party sharing:', userId);
  }

  async updateDataProcessingAgreements(userId) {
    console.log('Updating data processing agreements:', userId);
  }

  async deleteUserProfile(userId) {
    console.log('Deleting user profile:', userId);
  }

  async deleteUserActivity(userId) {
    console.log('Deleting user activity:', userId);
  }

  async deleteUserPreferences(userId) {
    console.log('Deleting user preferences:', userId);
  }

  async deleteMarketingData(userId) {
    console.log('Deleting marketing data:', userId);
  }

  async signOutUser() {
    console.log('Signing out user');
  }
}

export default new CCPACompliance();
```

---

## 🏥 **HIPAA Compliance**

### **HIPAA Key Requirements**
- **Privacy Rule**: Protects individual health information
- **Security Rule**: Sets security standards for electronic health information
- **Breach Notification Rule**: Requires notification of breaches
- **Enforcement Rule**: Provides for enforcement of the rules
- **Business Associate Agreements**: Required for business associates

### **HIPAA Implementation**
```javascript
// services/hipaaCompliance.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import CryptoJS from 'crypto-js';

class HIPAACompliance {
  constructor() {
    this.encryptionKey = process.env.HIPAA_ENCRYPTION_KEY;
    this.auditLogKey = 'hipaa_audit_log';
  }

  // Encrypt PHI (Protected Health Information)
  encryptPHI(data) {
    return CryptoJS.AES.encrypt(JSON.stringify(data), this.encryptionKey).toString();
  }

  // Decrypt PHI
  decryptPHI(encryptedData) {
    const bytes = CryptoJS.AES.decrypt(encryptedData, this.encryptionKey);
    return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
  }

  // Store encrypted health data
  async storeHealthData(userId, healthData) {
    try {
      const encryptedData = this.encryptPHI(healthData);

      const dataEntry = {
        userId,
        encryptedData,
        storedAt: new Date().toISOString(),
        storageLocation: 'encrypted_local_storage',
      };

      await AsyncStorage.setItem(`health_data_${userId}`, JSON.stringify(dataEntry));

      // Log access for audit
      await this.logAccess('store', userId, 'health_data');

      return { success: true };
    } catch (error) {
      console.error('Error storing health data:', error);
      await this.logAccess('store_failed', userId, 'health_data', { error: error.message });
      throw error;
    }
  }

  // Retrieve health data
  async getHealthData(userId) {
    try {
      const dataEntryJson = await AsyncStorage.getItem(`health_data_${userId}`);

      if (!dataEntryJson) {
        return null;
      }

      const dataEntry = JSON.parse(dataEntryJson);
      const healthData = this.decryptPHI(dataEntry.encryptedData);

      // Log access for audit
      await this.logAccess('retrieve', userId, 'health_data');

      return healthData;
    } catch (error) {
      console.error('Error retrieving health data:', error);
      await this.logAccess('retrieve_failed', userId, 'health_data', { error: error.message });
      throw error;
    }
  }

  // Log all access to PHI
  async logAccess(action, userId, resource, additionalData = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      action,
      userId,
      resource,
      ipAddress: await this.getClientIP(),
      userAgent: 'React Native Health App',
      ...additionalData,
    };

    try {
      const existingLogs = await AsyncStorage.getItem(this.auditLogKey);
      const logs = existingLogs ? JSON.parse(existingLogs) : [];

      logs.push(logEntry);

      // Keep only last 1000 entries to prevent storage bloat
      if (logs.length > 1000) {
        logs.splice(0, logs.length - 1000);
      }

      await AsyncStorage.setItem(this.auditLogKey, JSON.stringify(logs));
    } catch (error) {
      console.error('Error logging access:', error);
    }
  }

  // Get audit log
  async getAuditLog(userId = null, limit = 100) {
    try {
      const logsJson = await AsyncStorage.getItem(this.auditLogKey);
      if (!logsJson) return [];

      let logs = JSON.parse(logsJson);

      // Filter by user if specified
      if (userId) {
        logs = logs.filter(log => log.userId === userId);
      }

      // Sort by timestamp (newest first)
      logs.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

      // Limit results
      return logs.slice(0, limit);
    } catch (error) {
      console.error('Error retrieving audit log:', error);
      return [];
    }
  }

  // Handle breach notification
  async handleBreachNotification(breachDetails) {
    const notification = {
      type: 'breach',
      details: breachDetails,
      timestamp: new Date().toISOString(),
      affectedUsers: breachDetails.affectedUsers || [],
      breachType: breachDetails.breachType || 'unknown',
    };

    // Log the breach
    await this.logBreach(notification);

    // In a real implementation, this would:
    // 1. Notify affected individuals
    // 2. Notify HHS Office for Civil Rights
    // 3. Notify media outlets (if >500 individuals affected)
    // 4. Provide identity theft protection

    console.log('Breach notification processed:', notification);
  }

  // Log security breach
  async logBreach(breachDetails) {
    const breachLog = {
      ...breachDetails,
      loggedAt: new Date().toISOString(),
      regulation: 'HIPAA',
    };

    // Store breach log separately
    await AsyncStorage.setItem('hipaa_breaches', JSON.stringify([breachLog]));
  }

  // Implement minimum necessary access
  async checkAccessAuthorization(userId, resource, action) {
    // This would check user roles and permissions
    const userRole = await this.getUserRole(userId);

    const accessRules = {
      'health_data': {
        'patient': ['read'],
        'doctor': ['read', 'write'],
        'admin': ['read', 'write', 'delete'],
      },
    };

    const allowedActions = accessRules[resource]?.[userRole] || [];
    return allowedActions.includes(action);
  }

  // Business Associate Agreement validation
  async validateBusinessAssociate(associateId) {
    // This would validate that business associates have proper BAA in place
    const associates = await this.getBusinessAssociates();
    const associate = associates.find(a => a.id === associateId);

    if (!associate) {
      throw new Error('Business associate not found');
    }

    if (!associate.baaSigned) {
      throw new Error('Business Associate Agreement not signed');
    }

    if (new Date(associate.baaExpiry) < new Date()) {
      throw new Error('Business Associate Agreement expired');
    }

    return true;
  }

  // Risk analysis for data processing
  async performRiskAnalysis(dataProcessing) {
    const risks = [];

    // Check data sensitivity
    if (dataProcessing.includesPHI) {
      risks.push('PHI exposure risk');
    }

    // Check transmission method
    if (dataProcessing.transmissionMethod === 'unencrypted') {
      risks.push('Unencrypted transmission risk');
    }

    // Check storage method
    if (dataProcessing.storageMethod === 'unencrypted') {
      risks.push('Unencrypted storage risk');
    }

    // Check access controls
    if (!dataProcessing.hasAccessControls) {
      risks.push('Insufficient access controls');
    }

    const riskLevel = risks.length > 2 ? 'High' :
                     risks.length > 0 ? 'Medium' : 'Low';

    return {
      risks,
      riskLevel,
      recommendations: this.getRiskRecommendations(risks),
    };
  }

  // Get risk mitigation recommendations
  getRiskRecommendations(risks) {
    const recommendations = [];

    if (risks.includes('PHI exposure risk')) {
      recommendations.push('Implement additional encryption measures');
      recommendations.push('Limit PHI access to authorized personnel only');
    }

    if (risks.includes('Unencrypted transmission risk')) {
      recommendations.push('Use TLS 1.3 for all data transmissions');
      recommendations.push('Implement certificate pinning');
    }

    if (risks.includes('Unencrypted storage risk')) {
      recommendations.push('Encrypt data at rest using AES-256');
      recommendations.push('Implement proper key management');
    }

    if (risks.includes('Insufficient access controls')) {
      recommendations.push('Implement role-based access control (RBAC)');
      recommendations.push('Enable multi-factor authentication');
    }

    return recommendations;
  }

  // Generate compliance report
  async generateComplianceReport() {
    const auditLogs = await this.getAuditLog();
    const breaches = await this.getBreaches();

    const report = {
      generatedAt: new Date().toISOString(),
      period: {
        from: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString(), // 30 days ago
        to: new Date().toISOString(),
      },
      summary: {
        totalAccessEvents: auditLogs.length,
        totalBreaches: breaches.length,
        phiAccessEvents: auditLogs.filter(log => log.resource === 'health_data').length,
        unauthorizedAccessAttempts: auditLogs.filter(log => log.action.includes('failed')).length,
      },
      riskAssessment: await this.assessCurrentRisk(),
      recommendations: await this.generateRecommendations(),
    };

    return report;
  }

  // Assess current risk level
  async assessCurrentRisk() {
    const auditLogs = await this.getAuditLog();
    const breaches = await this.getBreaches();

    let riskScore = 0;

    // Increase risk for breaches
    riskScore += breaches.length * 10;

    // Increase risk for failed access attempts
    const failedAttempts = auditLogs.filter(log => log.action.includes('failed')).length;
    riskScore += failedAttempts * 2;

    // Increase risk for PHI access
    const phiAccess = auditLogs.filter(log => log.resource === 'health_data').length;
    riskScore += phiAccess * 0.5;

    const riskLevel = riskScore > 50 ? 'High' :
                     riskScore > 20 ? 'Medium' : 'Low';

    return {
      score: riskScore,
      level: riskLevel,
      factors: {
        breaches: breaches.length,
        failedAttempts,
        phiAccess,
      },
    };
  }

  // Generate recommendations
  async generateRecommendations() {
    const riskAssessment = await this.assessCurrentRisk();
    const recommendations = [];

    if (riskAssessment.level === 'High') {
      recommendations.push('Immediate security audit required');
      recommendations.push('Review and update access controls');
      recommendations.push('Implement additional monitoring');
    } else if (riskAssessment.level === 'Medium') {
      recommendations.push('Enhance access logging');
      recommendations.push('Review user permissions');
      recommendations.push('Update security policies');
    } else {
      recommendations.push('Continue current security measures');
      recommendations.push('Regular security training for staff');
    }

    return recommendations;
  }

  // Helper methods
  async getUserRole(userId) {
    // In a real implementation, this would check user roles from database
    return 'patient'; // Default role
  }

  async getBusinessAssociates() {
    // In a real implementation, this would return list of business associates
    return [
      {
        id: 'associate1',
        name: 'Cloud Storage Provider',
        baaSigned: true,
        baaExpiry: '2024-12-31',
      },
    ];
  }

  async getBreaches() {
    try {
      const breachesJson = await AsyncStorage.getItem('hipaa_breaches');
      return breachesJson ? JSON.parse(breachesJson) : [];
    } catch (error) {
      return [];
    }
  }

  async getClientIP() {
    // In production, get from server request
    return '192.168.1.1';
  }
}

export default new HIPAACompliance();
```

---

## 🔒 **Data Privacy Management**

### **Privacy Dashboard**
```javascript
// components/PrivacyDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import GDPRCompliance from '../services/gdprCompliance';
import CCPACompliance from '../services/ccpaCompliance';
import HIPAACompliance from '../services/hipaaCompliance';

const PrivacyDashboard = () => {
  const [privacyData, setPrivacyData] = useState({
    gdpr: null,
    ccpa: null,
    hipaa: null,
  });
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadPrivacyData();
  }, []);

  const loadPrivacyData = async () => {
    try {
      const [gdprData, ccpaData, hipaaData] = await Promise.all([
        GDPRCompliance.getConsentStatus(),
        CCPACompliance.hasSeenPrivacyNotice(),
        HIPAACompliance.getAuditLog(),
      ]);

      setPrivacyData({
        gdpr: gdprData,
        ccpa: ccpaData,
        hipaa: hipaaData?.length || 0,
      });
    } catch (error) {
      console.error('Error loading privacy data:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleGDPRRequest = async (requestType) => {
    try {
      switch (requestType) {
        case 'access':
          await GDPRCompliance.handleDataAccessRequest();
          Alert.alert('Success', 'Data access request submitted. You will receive a report via email.');
          break;
        case 'delete':
          await GDPRCompliance.handleDataDeletionRequest();
          Alert.alert('Success', 'Data deletion request processed.');
          break;
        case 'withdraw':
          await GDPRCompliance.withdrawConsent();
          Alert.alert('Success', 'Consent withdrawn.');
          break;
      }
    } catch (error) {
      Alert.alert('Error', 'Failed to process request. Please try again.');
    }
  };

  const handleCCPARequest = async (requestType) => {
    try {
      switch (requestType) {
        case 'doNotSell':
          await CCPACompliance.handleDoNotSellRequest();
          break;
        case 'delete':
          await CCPACompliance.handleDeletionRequest();
          break;
        case 'knowData':
          await CCPACompliance.handleKnowMyDataRequest();
          break;
      }
    } catch (error) {
      Alert.alert('Error', 'Failed to process request. Please try again.');
    }
  };

  if (loading) {
    return (
      <View style={styles.center}>
        <Text>Loading privacy data...</Text>
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Privacy Dashboard</Text>

      {/* GDPR Section */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>GDPR Rights</Text>
        <Text style={styles.sectionDescription}>
          As a user in the European Union, you have rights under GDPR.
        </Text>

        <View style={styles.buttonGroup}>
          <TouchableOpacity
            style={styles.button}
            onPress={() => handleGDPRRequest('access')}
          >
            <Text style={styles.buttonText}>Request Data Access</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[styles.button, styles.deleteButton]}
            onPress={() => handleGDPRRequest('delete')}
          >
            <Text style={styles.buttonText}>Delete My Data</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.button}
            onPress={() => handleGDPRRequest('withdraw')}
          >
            <Text style={styles.buttonText}>Withdraw Consent</Text>
          </TouchableOpacity>
        </View>

        {privacyData.gdpr && (
          <View style={styles.consentInfo}>
            <Text>Consent Status: {privacyData.gdpr.consent ? 'Granted' : 'Not Granted'}</Text>
            <Text>Last Updated: {new Date(privacyData.gdpr.timestamp).toLocaleDateString()}</Text>
          </View>
        )}
      </View>

      {/* CCPA Section */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>CCPA Rights</Text>
        <Text style={styles.sectionDescription}>
          As a California resident, you have rights under CCPA.
        </Text>

        <View style={styles.buttonGroup}>
          <TouchableOpacity
            style={styles.button}
            onPress={() => handleCCPARequest('knowData')}
          >
            <Text style={styles.buttonText}>Know My Data</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.button}
            onPress={() => handleCCPARequest('doNotSell')}
          >
            <Text style={styles.buttonText}>Do Not Sell My Data</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[styles.button, styles.deleteButton]}
            onPress={() => handleCCPARequest('delete')}
          >
            <Text style={styles.buttonText}>Delete My Data</Text>
          </TouchableOpacity>
        </View>

        <View style={styles.consentInfo}>
          <Text>Privacy Notice Seen: {privacyData.ccpa ? 'Yes' : 'No'}</Text>
        </View>
      </View>

      {/* HIPAA Section */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Health Data Privacy</Text>
        <Text style={styles.sectionDescription}>
          Your health information is protected under HIPAA.
        </Text>

        <View style={styles.auditInfo}>
          <Text>Audit Events: {privacyData.hipaa}</Text>
          <Text>Last 30 days of access logging</Text>
        </View>
      </View>

      {/* General Settings */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Privacy Settings</Text>

        <TouchableOpacity
          style={styles.button}
          onPress={() => Alert.alert('Privacy Policy', 'Opening privacy policy...')}
        >
          <Text style={styles.buttonText}>View Privacy Policy</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.button}
          onPress={() => Alert.alert('Cookie Settings', 'Opening cookie settings...')}
        >
          <Text style={styles.buttonText}>Cookie Settings</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    margin: 20,
  },
  section: {
    backgroundColor: 'white',
    margin: 10,
    borderRadius: 10,
    padding: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  sectionDescription: {
    fontSize: 14,
    color: '#666',
    marginBottom: 15,
  },
  buttonGroup: {
    gap: 10,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  deleteButton: {
    backgroundColor: '#dc3545',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  consentInfo: {
    marginTop: 15,
    padding: 10,
    backgroundColor: '#f8f9fa',
    borderRadius: 5,
  },
  auditInfo: {
    marginTop: 15,
    padding: 10,
    backgroundColor: '#e9ecef',
    borderRadius: 5,
  },
});

export default PrivacyDashboard;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **GDPR Compliance**: EU data protection regulation implementation
- ✅ **CCPA Compliance**: California consumer privacy rights
- ✅ **HIPAA Compliance**: Health information privacy and security
- ✅ **Data Privacy Management**: Privacy dashboard and user controls
- ✅ **Consent Management**: User consent collection and management
- ✅ **Audit & Logging**: Compliance event logging and monitoring
- ✅ **International Compliance**: Handling different regulatory jurisdictions
- ✅ **Compliance Automation**: Automated compliance checking and reporting

### **Best Practices**
1. **Privacy by Design**: Build privacy considerations into system architecture
2. **Data Minimization**: Collect only necessary data for legitimate purposes
3. **User Consent**: Obtain clear, informed consent for data processing
4. **Security Measures**: Implement appropriate technical and organizational security
5. **Regular Audits**: Conduct regular compliance audits and assessments
6. **Incident Response**: Have plans for handling data breaches and incidents
7. **User Rights**: Provide easy mechanisms for users