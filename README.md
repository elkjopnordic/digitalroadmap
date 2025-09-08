# Digital Roadmap Application

A static web application for visualizing eCommerce development roadmaps, built with Axure RP and deployed on Azure App Service.

## Overview

This is an interactive roadmap application that displays development plans, project timelines, and team allocations for eCommerce initiatives. The application is generated from Axure RP prototypes and exported as static HTML files with interactive JavaScript functionality.

## Application Structure

### Core Pages

- **`current.html`** - Current roadmap view (default page)
- **`index.html`** - Main entry point
- **`focus_time.html`** - Focus time allocations
- **`team.html`** - Team view
- **`ecom_dept.html`** - E-commerce department view
- **`parking_lot.html`** - Parking lot items
- **`ngrc.html`** - NGRC view
- **Historical views:**
  - `2017.html` - 2017 roadmap
  - `20191106-stu.html` - November 2019 study version
  - `currentold.html` - Previous current view

### Technology Stack

- **Frontend**: Axure RP generated HTML/CSS/JavaScript
- **Framework**: jQuery 1.7.1 + jQuery UI 1.8.10
- **Hosting**: Azure App Service
- **Authentication**: Intended for SSO (Azure AD B2B/B2C) - *Note: Authentication not currently implemented*
- **Deployment**: GitHub Actions CI/CD

## Project Structure

```text
digitalroadmap/
├── *.html                     # Main roadmap pages
├── web.config                  # IIS configuration
├── data/                       # Application data and assets
│   ├── document.js            # Main application configuration
│   ├── styles.css             # Global styles
│   └── apple-touch-icon.png   # Mobile icon
├── files/                      # Page-specific data and styles
│   ├── current/               # Current page assets
│   ├── team/                  # Team page assets
│   ├── focus_time/            # Focus time assets
│   └── [other-pages]/         # Other page-specific assets
├── images/                     # Static images organized by page
├── resources/                  # Axure framework resources
│   ├── css/                   # Axure CSS files
│   ├── scripts/               # Axure JavaScript framework
│   └── chrome/                # Browser compatibility files
├── plugins/                    # Axure plugins
└── .github/workflows/          # GitHub Actions deployment
```

## Deployment

### Azure App Service Configuration

The application is deployed to Azure App Service with the following configuration:

- **App Service Name**: `digroadmap`
- **Environment**: Production
- **Platform**: Windows
- **Default Document**: `current.html` (configured in `web.config`)

### GitHub Actions CI/CD

Automated deployment is configured via `.github/workflows/master_digroadmap.yml`:

```yaml
# Triggers on push to master branch
# Build: .NET Core (though this is a static site)
# Deploy: Azure Web Apps Deploy action
# Authentication: Azure Service Principal with OIDC
```

**Deployment Secrets Required:**

- `AZUREAPPSERVICE_CLIENTID_91CA4AAAC76B474C80534054C1B5FD99`
- `AZUREAPPSERVICE_TENANTID_A4B376FA7DD84223822A75099E803342`
- `AZUREAPPSERVICE_SUBSCRIPTIONID_A38383C862E14D25B0961AE0472A7FA0`

### Manual Deployment

To deploy manually:

1. Export the Axure RP project to HTML
2. Upload all files to Azure App Service via:
   - FTP/FTPS
   - Visual Studio
   - Azure CLI: `az webapp up`
   - Git deployment

## Development Workflow

### Modifying the Roadmap

1. **Open Axure RP Project**: Load the source `.rp` file in Axure RP
2. **Make Changes**: Update roadmap content, add/remove items, modify layouts
3. **Generate HTML**:
   - File → Publish → Generate HTML Files
   - Choose output directory
   - Ensure all pages are included
4. **Update Repository**:
   - Replace existing HTML files with new exports
   - Maintain folder structure (especially `files/`, `images/`, `resources/`)
   - Commit and push changes
5. **Deploy**: GitHub Actions will automatically deploy on push to `master`

### Key Files to Preserve

When updating from Axure exports:

- **Keep**: `web.config`, `.github/` folder, this `README.md`
- **Update**: All `.html` files, `files/`, `images/`, `data/`
- **Review**: `resources/` folder (may need updates from newer Axure versions)

## Testing Authentication & Deployment Status

### Quick Diagnosis Commands

You can test the current authentication and deployment status using these commands:

**1. Test Main Application URL:**

```powershell
curl -I https://digroadmap.azurewebsites.net/
```

Expected: Should return the application, but currently returns Azure default page

**2. Test Specific Application Files:**

```powershell
curl -I https://digroadmap.azurewebsites.net/current.html
curl -I https://digroadmap.azurewebsites.net/index.html
```

Expected: Should return 200 OK if files are deployed

**3. Check Content Type:**

```powershell
curl -s https://digroadmap.azurewebsites.net/ | Select-String -Pattern "title"
```

Look for: Application title vs "Microsoft Azure App Service - Welcome"

**4. Test Authentication Headers:**

```powershell
curl -v https://digroadmap.azurewebsites.net/ 2>&1 | Select-String -Pattern "Location:|WWW-Authenticate:|X-MS-"
```

Look for: Authentication redirects or Azure-specific headers

### Current Test Results

Based on testing performed on September 4, 2025:

- ✅ **Main URL accessible**: Returns HTTP 200 OK
- ❌ **Wrong content**: Serving Azure default welcome page instead of application
- ❌ **Application files missing**: `current.html` returns 404 Not Found
- ✅ **No authentication blocking**: No auth redirects or login prompts
- ⚠️ **SCM access restricted**: Requires basic authentication

### Diagnosis

**Issue**: Deployment problem, not authentication

- Application files are not properly deployed to the App Service
- The service is running but serving default Azure content
- Files may exist in SCM but not in the correct runtime location

## Authentication & Access Control

### Current State

⚠️ **Deployment Issue Identified**: The application has a deployment problem, not an authentication issue:

- **Main URL**: <https://digroadmap.azurewebsites.net/> serves Azure default welcome page
- **Application files**: Return 404 Not Found (not deployed to runtime location)
- **SCM Access**: <https://digroadmap.scm.azurewebsites.net/wwwroot/#g=1&p=current> works because it accesses files directly

**Root Cause**: Files are not properly deployed from the repository to the App Service runtime environment.

### Potential SSO Implementation

Based on the access pattern, SSO might already be configured at the Azure App Service level:

**Likely Authentication Methods:**

- **Azure App Service Authentication** (Easy Auth)
- **Azure Application Proxy** (for on-premises integration)
- **Azure AD integration** configured in the App Service settings
- **Custom authentication middleware**

**To verify current authentication setup:**

1. Check App Service Authentication settings in Azure Portal
2. Review any Azure AD app registrations named "digroadmap" or similar
3. Check if Application Proxy is configured for this service
4. Review App Service configuration for authentication providers

- **Azure AD B2B** for business users
- **Azure AD B2C** for customer authentication
- Integration points identified in roadmap items:
  - `B2X-3565 - B2B login to Azure`
  - `NEP-2755 - Next frontend for ADB2C auth for B2B`

### Implementing Authentication

To add Azure AD authentication:

1. **Azure AD App Registration**:

   ```bash
   az ad app create --display-name "Digital Roadmap"
   az ad app update --id <app-id> --web-redirect-uris "https://digroadmap.azurewebsites.net/"
   ```

2. **App Service Authentication**:

   ```bash
   az webapp auth update --resource-group <rg> --name digroadmap \
     --enabled true --action LoginWithAzureActiveDirectory \
     --aad-client-id <client-id> --aad-client-secret <secret>
   ```

3. **Update `web.config`** to include authentication requirements

## Browser Compatibility

The application includes compatibility files for:

- Chrome (requires extension for local files)
- Firefox (local file restrictions)
- Safari (local file restrictions)
- Mobile browsers (responsive design)

## Maintenance

### Regular Tasks

- **Content Updates**: Use Axure RP to modify roadmap content
- **Dependency Updates**: Monitor jQuery and Axure framework updates
- **Access Review**: Review Azure App Service access logs
- **Performance**: Monitor page load times and optimize images

### Additional Azure CLI Diagnostic Commands

If you have Azure CLI access:

```bash
# Check deployment status
az webapp deployment list --name digroadmap --resource-group <rg-name>

# Check app settings
az webapp config appsettings list --name digroadmap --resource-group <rg-name>

# Check deployment source
az webapp deployment source show --name digroadmap --resource-group <rg-name>

# Check logs
az webapp log tail --name digroadmap --resource-group <rg-name>
```

### Troubleshooting

**Current Access Issue:**

The main application URL is not working. To diagnose:

1. **Check Azure App Service Authentication**:

   ```bash
   az webapp auth show --name digroadmap --resource-group <rg-name>
   ```

2. **Verify file deployment**:
   - Access Kudu: <https://digroadmap.scm.azurewebsites.net/>
   - Check if files are in `/site/wwwroot/`
   - Verify `current.html` exists and is accessible

3. **Check App Service Configuration**:

   ```bash
   az webapp config show --name digroadmap --resource-group <rg-name>
   ```

4. **Review deployment logs**:
   - Check GitHub Actions deployment logs
   - Review App Service deployment center logs

**Common Issues:**

1. **Broken Links**: Check file paths in exported HTML
2. **Missing Images**: Ensure `images/` folder structure is preserved
3. **JavaScript Errors**: Verify `resources/scripts/` files are intact
4. **Styling Issues**: Check `data/styles.css` and page-specific CSS files

**Logs & Monitoring:**

- Azure App Service logs: Available in Azure Portal
- Application logs: Check browser console for JavaScript errors
- Deployment logs: Available in GitHub Actions

## Security Considerations

- **Static Content**: No server-side processing reduces attack surface
- **HTTPS**: Enforced by Azure App Service
- **Authentication**: Currently public; implement Azure AD for access control
- **Content Security**: Roadmap may contain sensitive business information

## Links & References

- **Live Application**: ⚠️ <https://digroadmap.azurewebsites.net/> (Currently not working)
- **Working Access**: <https://digroadmap.scm.azurewebsites.net/wwwroot/#g=1&p=current> (SCM endpoint)
- **Azure Portal**: Navigate to App Service "digroadmap"
- **Kudu/SCM Portal**: <https://digroadmap.scm.azurewebsites.net/>
- **GitHub Repository**: Current repository for source code
- **Axure RP**: <https://www.axure.com/> (for editing roadmap content)

## Support

For technical issues:

1. Check GitHub Issues in this repository
2. Review Azure App Service diagnostics
3. Contact the development team for Axure RP modifications
4. For authentication setup, consult Azure AD documentation

---

## TODO: Fix Deployment Issues

### 🚨 Critical Issues to Resolve

**Primary Issue**: Application files are not properly deployed to Azure App Service runtime.

### Required Actions

#### 1. Fix GitHub Actions Workflow

The current workflow (`.github/workflows/master_digroadmap.yml`) is configured for .NET applications but this is a static HTML site.

**Changes needed:**

- Remove .NET build steps (`dotnet build`, `dotnet publish`)
- Replace with direct file copy/upload
- Ensure files are deployed to correct App Service location

#### 2. Update Deployment Process

**Option A: Use Azure Static Web Apps** (Recommended)

- Migrate from App Service to Static Web Apps
- Better suited for Axure-generated static content
- Simpler deployment process

#### Option B: Fix Current App Service Deployment

- Update GitHub Actions to use `azure/webapps-deploy@v3` with static files
- Ensure proper file structure deployment
- Verify `web.config` is properly deployed

#### 3. Verify Default Document Configuration

- Confirm `web.config` sets `current.html` as default document
- Test that Azure App Service respects the configuration
- Consider adding `index.html` redirect to `current.html`

#### 4. Test and Validate

After deployment fixes:

- Verify <https://digroadmap.azurewebsites.net/> serves the application
- Test all roadmap pages load correctly
- Confirm navigation between pages works
- Validate mobile responsiveness

### Implementation Priority

1. **High**: Fix GitHub Actions deployment workflow
2. **Medium**: Verify web.config default document settings
3. **Low**: Consider migration to Static Web Apps for better performance

---

*Last Updated: September 2025*  
*Application Version: Based on Axure RP exports*
