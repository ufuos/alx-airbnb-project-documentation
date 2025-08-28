    <!-- System boundary -->
    <mxCell id="sys" value="Airbnb Clone Backend" style="rounded=1;whiteSpace=wrap;html=1;fillOpacity=0;strokeWidth=2;" vertex="1" parent="1">
      <mxGeometry x="120" y="60" width="920" height="700" as="geometry"/>
    </mxCell>

    <!-- Use cases (ovals) inside system boundary -->
    <mxCell id="uc_user_mgmt" value="User Management\n(Register, Login, Manage Profile, Roles)" style="ellipse;whiteSpace=wrap;html=1;container=0;" vertex="1" parent="1">
      <mxGeometry x="200" y="100" width="260" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_property" value="Property Management\n(Create/Edit/Delete, Upload Images, Manage Availability)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="520" y="100" width="360" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_search_booking" value="Search & Booking\n(Search, Filter, Create Booking, Cancel)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="200" y="240" width="360" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_payments" value="Payments\n(Make Payment, Refund, Host Payout)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="600" y="240" width="280" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_reviews" value="Reviews & Ratings\n(Leave Review, Host Responds, Aggregate)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="200" y="380" width="360" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_notifications" value="Notifications\n(Receive Booking/Payment Notices, Email Confirmations)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="600" y="380" width="360" height="80" as="geometry"/>
    </mxCell>
    <mxCell id="uc_admin" value="Admin Controls\n(Manage Users/Listings/Bookings, Refunds, Reports)" style="ellipse;whiteSpace=wrap;html=1;" vertex="1" parent="1">
      <mxGeometry x="360" y="520" width="480" height="80" as="geometry"/>
    </mxCell>

    <!-- Connectors from actors to use cases -->
    <mxCell id="c_guest_user" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_guest" target="uc_search_booking">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>
    <mxCell id="c_guest_payment" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_guest" target="uc_payments">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>
    <mxCell id="c_guest_reviews" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_guest" target="uc_reviews">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

    <mxCell id="c_host_property" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_host" target="uc_property">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>
    <mxCell id="c_host_booking" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_host" target="uc_search_booking">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>
    <mxCell id="c_host_reviews" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_host" target="uc_reviews">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

    <mxCell id="c_admin_all" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;dashed=1;" edge="1" parent="1" source="a_admin" target="uc_admin">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

    <mxCell id="c_payment_payments" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_payment" target="uc_payments">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

    <mxCell id="c_email_notifications" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_email" target="uc_notifications">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

    <!-- Connections for user management -->
    <mxCell id="c_guest_user_mgmt" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_guest" target="uc_user_mgmt">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>
    <mxCell id="c_host_user_mgmt" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;" edge="1" parent="1" source="a_host" target="uc_user_mgmt">
      <mxGeometry relative="1" as="geometry"/>
    </mxCell>

  </root>
</mxGraphModel>
]]>
