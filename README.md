# Link unfurling samples v2

## Email Footer Customization

The email footer is **not** a single editable text area. It is composed from a set of discrete, independently configurable blocks. Understanding this composition model is essential for customizing your footer correctly.

### Footer Blocks

The footer is assembled from the following blocks, each of which is configured separately:

| Block | Description |
|---|---|
| **Address** | Your organization's mailing address, required for CAN-SPAM / GDPR compliance. |
| **Disclaimer** | Legal or regulatory disclaimer text appended beneath the address. |
| **Social Links** | Icons and URLs for your social-media profiles (e.g. Twitter, LinkedIn, Facebook). |

### How to Customize Each Block

1. Navigate to **Settings → Email → Footer**.
2. Select the specific block you want to edit (Address, Disclaimer, or Social Links).
3. Update the content for that block and save.

Repeat for each block you need to change. There is **no single textarea** that controls the entire footer — every block must be edited individually.

### Frequently Asked Questions

**Why can't I edit the whole footer at once?**
The block-based model allows each component (legal address, disclaimers, social links) to be managed, translated, and permissioned independently. This is by design, not a limitation.

**A previous bug report said the footer textarea was missing — is this a bug?**
No. There is no single footer textarea in the product. Previous reports on this topic were closed as non-product issues. Footer customization is done block-by-block as described above.
