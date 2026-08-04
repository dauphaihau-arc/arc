# Access Control

Arc uses one account system across marketplace surfaces, but each portal has its own role gate. Portal access must be based on roles, not broad API permissions.

## Portal Access Matrix

| Portal | Customer | Seller | Admin |
| --- | --- | --- | --- |
| Storefront | Allowed | Allowed | Blocked |
| Seller Portal | Blocked | Allowed | Blocked |
| Admin Portal | Blocked | Blocked | Allowed |

## Role Rules

### Customer

- Can browse products, manage cart, place orders, view account data, and message sellers.
- Cannot manage shops, products, seller orders, users, roles, or platform settings.
- Cannot access the seller portal unless also assigned the seller role.
- Cannot access the admin portal.

### Seller

- Can access the seller portal to manage their own shop, products, inventory, promotions, orders, and buyer messages.
- Can also access the storefront as a customer to browse and buy from other sellers.
- Cannot access the admin portal.

### Admin

- Can access only the admin portal.
- Cannot directly log into the storefront or seller portal.
- Should use explicit impersonation for support workflows when that feature exists.
- Admin portal is internal and must not be linked from storefront or seller UI.

## Portal Entry Rules

- Storefront login accepts accounts with `customer` or `seller` roles.
- Seller portal login accepts accounts with the `seller` role.
- Admin portal login accepts accounts with the `admin` role.
- If an account includes the `admin` role, treat it as admin-only for portal access.

## Navigation Rules

- Storefront may link to seller registration or seller portal entry points.
- Seller portal may link back to storefront pages such as the public shop or product pages.
- Storefront and seller portal must not link to the admin portal.
- Customer-facing copy should not expose internal admin tooling names.

## Permissions Versus Portal Access

Permissions authorize API capabilities after a user has entered an allowed portal. They must not decide portal login.

For example, `shops.manage` can authorize shop-management API routes, but it should not by itself allow seller portal login. Seller portal login should check the `seller` role.

## Seed Account Expectations

Seeded accounts should follow the same rules:

- `admin@example.com` should only be usable for the admin portal.
- Seller demo accounts should be usable in the seller portal and storefront.
- Customer demo accounts should be usable in the storefront only.

## Future Impersonation

Admin support workflows should use explicit impersonation rather than direct admin login to storefront or seller portal.

When implemented, impersonation should:

- Require an admin-authenticated session from the admin portal.
- Record the acting admin, target user, start time, end time, and reason.
- Make the impersonated state visible in the UI.
- Avoid exposing admin portal links from customer or seller surfaces.
