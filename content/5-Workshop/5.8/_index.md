---
title: "Amazon Location GeoPlaces V2"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

After the event pipeline is stable, replace Nominatim with Amazon Location so the entire core flow uses target services before the final end-to-end test.

5.8.1. Integration Design
The backend uses the `AWSSDK.GeoPlaces` package and an IAM Role to call SearchText/ReverseGeocode.
The browser only calls the internal APIs `/api/location/search` and `/api/location/reverse`; it does not receive AWS credentials.
The API returns a stable model containing `displayName`, `title`, `latitude`, and `longitude` so the UI does not depend directly on the SDK response.
Do not silently fall back to Nominatim after the cutover; if a rollback is needed, use an explicit feature flag and log the provider.

5.8.2. IAM and Environment Variables
```env
AmazonLocation__Region=ap-southeast-1
AWS_REGION=ap-southeast-1
```
```json
{
  "Effect": "Allow",
  "Action": [
    "geo-places:SearchText",
    "geo-places:ReverseGeocode",
    "geo-places:GetPlace",
    "geo-places:Suggest"
  ],
  "Resource": "arn:aws:geo-places:ap-southeast-1::provider/default"
}
```

5.8.3. CLI and Application Testing
```bash
sudo -u deliveryapp aws geo-places search-text   --query-text "BHD Star Lê Văn Việt, Thành phố Hồ Chí Minh"   --bias-position 106.6297 10.8231   --filter 'IncludeCountries=VNM'   --max-results 3   --language vi   --intended-use Storage   --region ap-southeast-1

sudo -u deliveryapp aws geo-places reverse-geocode   --query-position 106.778936 10.845672   --max-results 3   --language vi   --intended-use Storage   --region ap-southeast-1
```
1. Log in and open the order creation page.
2. Enter an address, call `/api/location/search?query=...` and verify HTTP 200.
3. Select a result; latitude/longitude and the map marker must update.
4. Click the map; `/api/location/reverse` must auto-fill the address.
5. Save the order and check the Lat/Lng stored in the database.

Evidence to add: Screenshots of the Network response for the two internal APIs, the updated map marker after selecting an address, and the database record showing the order coordinates. Do not screenshot AWS credentials since the browser should not receive them.